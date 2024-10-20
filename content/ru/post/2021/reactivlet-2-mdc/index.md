---
title: "От сервлетов к реактору 2"
subtitle: "Часть 2. Универсальная поддержка MDC"
summary: "Часть 2. Универсальная поддержка MDC"
authors:
  - toparvion
tags:
  - spring
  - reactor
  - servlet
  - webmvc
  - webflux
  - mdc
  - logging
categories:
  - Performance
series:
  - reactivlet
date: 2021-12-10T08:31:05+07:00
# lastmod: 2021-12-08T07:32:05+07:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "[Реакторный зал Белоярской АЭС](#о-картинке)"
  focal_point: ""
  preview_only: false

links:
  - icon: book
    icon_pack: fas
    name: Серия статей ReactivLet
    url: /series/reactivlet/
  - icon: github
    icon_pack: fab
    name: Код примеров
    url: https://github.com/Toparvion/reactivlet-sample

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

{{< toc >}}

Одна из самых популярных задач, взрывающих мозг честным людям при переходе с сервлетного стека на реактивный, – это обеспечение повсеместного вывода [MDC-меток](http://slf4j.org/manual.html#mdc) в логах приложения.

{{< spoiler text="Что такое MDC?" >}}

Это строковые диагностические метки, которые позволяют легко находить в логах записи, относящиеся к одному этапу/аспекту/модулю приложения. Хрестоматийный пример: идентификатор пользовательского запроса (request id, `rid`) – позволяет выявить в логе все записи, относящиеся к обработке каждого входящего запроса, из какой бы части приложения эти записи не были сделаны.

{{< /spoiler >}}

## Проблема

В классических сервлетных приложениях эта задача решалась относительно просто за счёт того, что каждому входящему запросу, как правило, соответствовал **один** поток-обработчик в пуле сервлет-контейнера. Благодаря этому, оснастка логирования могла переложить значение, например, `rid` из HTTP-заголовка или параметров в какую-либо поточно-локальную переменную (`ThreadLocal`), а потом обращаться к ней как к статической переменной из любого места приложения, зная, что в текущем потоке эта переменная теперь будет возвращать одно и то же значение (пока ее не очистят после отправки ответа клиенту). Упрощенно это могло бы выглядеть примерно так:

```java
public class ServletMdcFilter implements Filter {    // [1]
  @Override
  public void doFilter(ServletRequest request,
                       ServletResponse response,
                       FilterChain chain) throws ServletException, IOException {
    HttpServletRequest httpServletRequest = (HttpServletRequest) request;
    String rid = httpServletRequest.getParameter("rid");
    MDC.put("rid", rid);                             // [2]
    try {
      chain.doFilter(request, response);
    } finally {
      MDC.remove("rid");
    }
  }
}
```

:one: Класс `Filter` импортирован из `javax.servlet.Filter`.  
:two: Здесь под вызовом `put` скрывается обращение к поточно-локальной map’е. Например, в имплементации `org.slf4j.helpers.BasicMDCAdapter` она объявлена так:

```java
private InheritableThreadLocal<Map<String, String>> inheritableThreadLocal
```

Однако с переходом на стек реактивный, вся эта прелесть работать перестала. И дело не только в том, что принцип “один запрос – один поток” перестал соблюдаться, но и в том, что фильтры из Servlet API по понятным причинам вообще перестали как-либо учитываться и всё происходящее в них игнорируется.

## Вывод решения

{{< spoiler text="*Примечание для буквоедов*" >}}

Эрудированный читатель наверняка знает, что задаче проброса MDC-меток в документации на Project Reactor посвящен [отдельный пункт в FAQ](https://projectreactor.io/docs/core/release/reference/#faq.mdc). Почему бы просто не последовать ему? Потому что он предполагает написание **новой** логики в реактивном стиле, в то время как здесь решается задача адаптации **существующей** (императивной) логики к работе в реактивном окружении.

{{< /spoiler >}}

#### Короче (TL;DR)

Ради полноты понимания здесь будет изложен весь ход получения финального решения. Однако для страждущих результата “прям ща” предусмотрен короткий путь – [готовые классы](https://github.com/Toparvion/reactivlet-sample/tree/main/shared/src/main/java/pro/toparvion/sample/reactivlet/shared) `ReactiveMdcFilter` и `ServletMdcFilter` в прилагаемом демо-проекте{{< icon name="github" pack="fab" >}}.

### Переход на WebFilter

На первый взгляд может показаться, что “мигрировать” проброс MDC-меток с сервлетного стека на реактивный очень просто – достаточно поменять приведенный выше фильтр на соответствующий реактивный аналог `org.springframework.web.server.WebFilter`. Его основное отличие состоит лишь в том, что метод `filter` возвращает уже не чистый `void`, а обёртку `Mono<Void>`. Ну, и пара “запрос/ответ” собрана в один объект `exchange`. В остальном смысл, вроде как, тот же, так что переписать не сложно:

```java
public class ReactiveMdcFilter implements WebFilter {
  private static final Logger log = LoggerFactory.getLogger(ReactiveMdcFilter.class);
  @Override
  public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
    String rid = exchange.getRequest().getQueryParams().getFirst("rid");
    MDC.put("rid", rid);
    log.trace("Выставлена метка rid={}", rid);    // [1]
    try {
      return chain.filter(exchange);              // [2]
    } finally {
      MDC.remove("rid");
      log.trace("Убрана метка rid={}", rid);      // [1]
    }
  }
}
```

:one: Добавим чуток логирования для ясности происходящего.  
:two: Раньше здесь не было `return`, и выход из метода был просто в конце тела. Но ведь у нас используется `finally`, а значит, его логика выполнится по-любому, каким бы путем мы не выходили из `try {}`. Короче, всё должно быть хорошо.

Однако, если применить такой фильтр и отправить приложению запрос с URL-параметром `rid=123`, поведение окажется, мягко говоря, неожиданным:

```text
17:34:09.710 rid:123 TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Выставлена метка rid=123
17:34:09.710 rid:    TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Убрана метка rid=123
17:34:09.751 rid:     INFO --- [ctor-http-nio-3] p.t.s.g.g.GatewayDemoWebFluxApplication  : Перенаправляю запрос...
```

, то есть метка вроде как успешно поставилась и даже один раз отсветила в логе, но в то же мгновение зачем-то самоуничтожилась, оставив последующую прикладную логику ни с чем. При этом поток не менялся и вообще ничего подозрительного вокруг не происходило.

Дело вот в чём. Во-первых, вызов `return chain.filter(exchange)` – это пример реактивного [конвейера](https://projectreactor.io/docs/core/release/reference/#_the_assembly_line_analogy) (набора действий в реактивном стиле). Обычно такие конвейеры включают в себя какие-либо действия над пропускаемыми через него данными, и эти действия часто задаются лямбда-выражениями или ссылками на методы. Здесь ни того, ни другого нет, но это не значит, что нет их и дальше в цепочке фильтров, которой мы делегируем управление. Во-вторых, реактивные конвейеры выполняются как бы в два прохода. На первом проходе хоть и выполняется код, но он *не запускает* обработку данных в конвейере, а лишь *декларирует* его, т.е. описывает, из каких элементов он состоит, и как эти элементы связаны друг с другом. Сами действия над данными откладываются за счёт упомянутых выше ленивых конструкций типа лямбд и ссылок. И больше ничего не произойдёт до тех пор, пока у конвейера не [появится подписчик](https://projectreactor.io/docs/core/release/reference/#reactive.subscribe). Появиться он может только за счёт возвращаемого хвостика `Mono<>`  или `Flux<>`, у которых есть метод `subscribe`. И вот когда появится, это и будет второй проход: начнут выполняться декларированные ранее действия.

Так вот в нашем случае получается, что очистка MDC-метки в блоке `finally` честно отрабатывает еще на *первом* проходе, когда конвейер только строится, т.е. до того, как по нему пошли данные HTTP-запроса.

### Прививка реактивности

 Чтобы это исправить, нужно перенести оба действия с меткой на *второй* проход. Выставление метки в таком случае будет лучше сделать в момент подписки – для этого есть метод `doOnSubscribe`. А удаление – после завершения обработки, причём независимо от результата (для этого есть метод с внезапным именем `doFinally`):

```java
public class ReactiveMdcFilter implements WebFilter {
  private static final Logger log = LoggerFactory.getLogger(ReactiveMdcFilter.class);
  @Override
  public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
    String rid = exchange.getRequest().getQueryParams().getFirst("rid");
    return chain.filter(exchange)        // [1]
      .doOnSubscribe(subscription -> {   // [2]
        MDC.put("rid", rid);
        log.trace("Выставлена метка rid={}", rid);
      })
      .doFinally(whatever -> {           // [3]
        MDC.remove("rid");
        log.trace("Убрана метка rid={}", rid);
      });
  }
}
```

:one: Даём последующим фильтрам возможность дополнить цепочку своими звеньями. Здесь важно понимать, что это ещё не обработка запроса, как было бы в традиционной сервлетной цепочке, а лишь декларация новых звеньев. Именно поэтому мы можем описывать дополняемые нами действия *после* того, как этот метод вернёт управление.  
:two: Здесь `subscription` – это экземпляр `org.reactivestreams.Subscription`, но нам он пока не интересен.  
:three: Здесь `whatever ` – это элемент перечисления `reactor.core.publisher.SignalType` – тип сигнала, с которым завершает свою работу конвейер, но он нам пока тоже не интересен.

Будет ли такая конструкция работать? Конечно!

```text
11:07:12.963 rid:123 TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Выставлена метка rid=123
11:07:13.010 rid:123  INFO --- [ctor-http-nio-3] p.t.s.g.g.GatewayDemoWebFluxApplication  : Перенаправляю запрос...
11:07:13.231 rid:    TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Убрана метка rid=123
```

… но только до тех пор, пока вся обработка запроса (и построение конвейера, и его выполнение) остаётся в одном потоке. По умолчанию Reactor не будет плодить потоки или как-либо переключать их без ведома разработчика. Однако одна из его сильнейших сторон как раз и состоит в том, чтобы это делать, причем с минимальными усилиями разработчика. А понадобиться это может, например, для того, чтобы вынести какой-либо старый синхронный (еще пока не реактивный) код в отдельный поток, взятый из специального пула (не конкурирующего с основным). В Project Reactor это [делается](https://projectreactor.io/docs/core/release/reference/#faq.wrap-blocking) буквально одним оператором `subscribeOn(Schedulers.boundedElastic())`, и в рассматриваемом примере приведёт вот к чему:

```text
12:34:08.192 rid:123 TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Выставлена метка rid=123
12:34:08.238 rid:     INFO --- [oundedElastic-1] p.t.s.g.g.GatewayDemoWebFluxApplication  : Синхронно перенаправляю запрос...
12:34:08.332 rid:    TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Убрана метка rid=123
```

Здесь видно, что прикладная логика выполняется в потоке `boundedElastic-1`, отличном от исходного `reactor-http-nio-3`, поэтому поточно-локальная переменная недоступна, и MDC-метка `rid` не выведена. В силу врождённой асинхронности и неблокируемости, Reactor может жонглировать потоками, нарушая некогда непреложный принцип “один запрос – один поток”. Однако это же способность является его сильной стороной, и с ней нужно уметь уживаться.

### Дополнение

 До выхода Project Reactor версии 3.3 у этой задачи было только весьма [разлапистое](https://ttddyy.github.io/mdc-with-webclient-in-webmvc/#spring-boot-21-reactor-32) решение. Однако с выходом v3.3 появилась возможность декорировать задачи, раздаваемые [планировщикам](https://projectreactor.io/docs/core/release/reference/#schedulers) на выполнение. Это делается при помощи т.н. хуков – методов, принимающих исходный `Runnable`, и возвращающих его декорированный вариант. Благодаря такому подходу, они могут назначать действия для выполнения как в декорирующем потоке, так и в целевом. В том же классе `ReactiveMdcFilter` выглядеть это может примерно так:

```java
  @PostConstruct                          // [1]
  void setupThreadsDecorator() {
    Schedulers.onScheduleHook("mdc", runnable -> {
      String rid = MDC.get("rid");        // [2]
      return () -> {
        MDC.put("rid", rid);              // [3]
        log.trace("Метка 'rid' перенесена в поток");
        try {
          runnable.run();
        } finally {
          MDC.remove("rid");              // [4]
          log.trace("Метка 'rid' отвязана от потока");
        }
      };
    });
  }
  @PreDestroy                             // [5]
  void shutdownThreadsDecorator() {
    Schedulers.resetOnScheduleHook("mdc");
  }
```

:one: Говорим Spring’у, что хотим позвать этот метод после того, как содержащий его бин будет создан и проинициализирован самим фреймворком. Здесь неявно предполагается, что этот бин будет синглтоном (что по умолчанию в Spring’е уже так).  
:two: Извлекаем текущее значение метки. Под текущим понимается то, которые было выставлено описанным ранее фильтром. Такая связность обеспечивается тем, что именно эта часть декоратора выполняется еще в *исходном*, а не ответвленном потоке.  
:three: Кладём извлечённое ранее значение метки в то же самое место, но уже в *ответвлённом* потоке.  
:four: Очищаем метку безотносительно к тому, чем закончилось выполнение задачи. Это важно потому, что поток скоро вернётся в пул и может быть снова выдан кому-нибудь в работу. Если не очистить метку, и новый “хозяин” не перезатрёт её своей, читатель логов сойдёт с ума, пытаясь понять, почему у него в одну выборку попадают записи от абсолютно не связанных между собой действий.  
:five: Удаляем назначенный хук по выбранному ранее условному имени `mdc`. В большинстве нормальных программ это произойдёт лишь перед самым выключением, т.е. в принципе этого можно и не делать. Но в некоторых случаях (например, при горячей перезагрузке контекста) такой клининг всё же может быть полезным.

Теперь, даже несмотря на оборачивание синхронного кода в отдельный поток, MDC-метка всё равно корректно выводится на каждом шаге:

```text
14:17:07.803 rid:123 TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Выставлена метка rid=123
14:17:07.861 rid:123 TRACE --- [oundedElastic-1] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Метка 'rid' перенесена в этот поток
14:17:07.861 rid:123  INFO --- [oundedElastic-1] p.t.s.g.g.GatewayDemoWebFluxApplication  : Синхронно перенаправляю запрос...
14:17:07.945 rid:    TRACE --- [oundedElastic-1] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Метка 'rid' отвязана от этого потока
14:17:07.955 rid:    TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Убрана метка rid=123
```

Справедливости ради надо отметить, что таким решением можно охватить только те потоки, которые ответвляются реактивными планировщиками, а это, увы, не все случаи жизни. Однако на первых порах для адаптации сервлетной логики к реактивным реалиям этого должно быть достаточно.

### Двойное гражданство

Написанный нами фильтр теперь будет вполне естественно чувствовать себя в реактивной среде. Но что если этот фильтр окажется/является частью общего модуля (библиотеки), который может быть подключен как реактивному, так и к сервлетному приложению? Очевидно, что с сервлетным он уже не заработает (хотя бы потому, что имплементации `WebFilter` не будут никем вызваны).

Одно из решений состоит в том, чтобы сделать новый реактивный фильтр не заменой, а альтернативным дополнением к сервлетному (подобно отношениям между WebFlux и WebMVC), то есть чтобы существовали оба, но использовался только один.  Этого можно достичь использованием встроенной в Spring Boot аннотации `@ConditionalOnWebApplication`, которая на примере *реактивного* фильтра может выглядеть так:

```java
@Component
@ConditionalOnWebApplication(type = REACTIVE)           // [1]
public class ReactiveMdcFilter implements WebFilter {
```

:one: Здесь `REACTIVE` – это сокращённое заклинание `import static org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication.Type.REACTIVE;`

Соответственно, для *сервлетного* стека та же аннотация может выглядеть так:

```java
@Component
@ConditionalOnWebApplication(type = SERVLET)                // [1]
public class ServletMdcFilter extends GenericFilterBean {   // [2]
```

:one: Здесь может возникнуть соблазн не указывать эту аннотацию вовсе, ведь мы рассматриваем только два варианта: реактивный и сервлетный. Однако так лучше не делать, потому что (1) такое указание делает намерения разработчика более явными, а значит, и код – более предсказуемым; (2) с точки зрения Spring Boot есть ещё третий тип приложений – условно говоря, “никакой”, это когда никакого веб-сервера нет вообще (например, в CLI-приложении или sevrerless unit). На такие случаи может реагировать специальная аннотация `ConditionalOnNotWebApplication`.  
:two: `GenericFilterBean` – это Spring’овая адаптация упоминавшегося ранее интерфейса `javax.servlet.Filter`, просто она умеет чуть больше (имплементирует всего-то 7 интерфейсов).

В таком виде в приложении появляется “подстилка” под оба режима работы – и сервлетный, и реактивный, но каждая “оживает” только в своём режиме и не мешается в другом. Однако важно отметить, что поскольку MDC в этом подходе выставляется *лишь раз*, он **может некорректно работать** в случае *множества* событий, курсирующих между клиентом и сервером (например, WebSocket или SSE). Спасибо [@OlehDokuka](https://twitter.com/OlehDokuka){{< icon name="twitter" pack="fab" >}} за это ценное примечание!

Полные тексты обоих фильтров приведены в пакете [shared](https://github.com/Toparvion/reactivlet-sample/tree/main/shared/src/main/java/pro/toparvion/sample/reactivlet/shared) прилагаемого демо-проекта{{< icon name="github" pack="fab" >}}.

## Попутное резюме

Эта часть заметок была призвана вооружить читателя новой техникой обобщения логики между стеками. Имея её в арсенале, можно подступиться к более сложной задаче, например, сделать текущий HTTP-запрос повсеместно доступным независимо от стека. Этим и займётся [следующая]({{< relref "/post/2021/reactivlet-3-request" >}}) заметка.

&nbsp;

---

###### О картинке
<sup>
<a href="https://commons.wikimedia.org/wiki/File:RIAN_archive_895485_Beloyarsk_nuclear_power_plant_in_Sverdlovsk_Region.jpg">RIA Novosti archive, image #895485 / Pavel Lysizin / CC-BY-SA 3.0</a>, <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0</a>, через Викисклад
</sup>
