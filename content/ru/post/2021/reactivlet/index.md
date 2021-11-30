---
title: "Заголовок"
subtitle: "Подзаголовок"
summary: "Тизер"
authors:
  - toparvion
tags:
  - spring
  - reactor
  - servlet
categories:
  - РеактивныеСервлеты
date: 2020-04-14T08:31:05+07:00
lastmod: 2020-045-16T07:32:05+07:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

{{< toc >}}

Когда рассказывают о прелестях реактивного фреймворка [Spring WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux) и его подкапотном [Project Reactor](https://projectreactor.io/), для примера чаще всего показывают новые, создаваемые с нуля приложения. Однако на практике приходится строить из готовых блоков, в том числе собственных прикладных и инфраструктурных модулей, которые написаны в императивном стиле и опираются на сервлетный стек. Как правило, такие модули нельзя/некогда/не охота *(нужное подчеркнуть)* переписывать, поэтому надо как-то адаптировать их к реактивному миру с минимумом правок (а лучше без них вовсе). О некоторых подходах к этой задаче и пойдёт речь дальше. 

{{< spoiler text="Вместо disclaimer’а" >}}

Всё сказанное здесь опирается лишь на небольшой опыт автора в решении одной практической задачи, поэтому статья не претендует на полноту охвата темы ни вширь, ни вглубь. Её цель – обозначить **некоторые** проблемы и **возможные** решения для них.

{{< /spoiler >}}

## Анализ проблемы

{{% callout info %}} 

**Историческая справка**

Появившийся в Spring 5 реактивный веб-фреймворк WebFlux был [задуман](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/web-reactive.html#webflux) как невытесняющая альтернатива старому доброму [WebMVC](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/web.html#spring-web), т.е. существуют и развиваются они оба, но в качестве серверного стека можно использовать только один. Дело не столько в вездесущих обёртках `Mono<>` и `Flux<>` вокруг результатов чуть ли не всех методов, сколько в принципиально иной модели многопоточности под капотом.

 {{% /callout %}}

Поскольку большинство сервлетных Spring-приложений построено на WebMVC (в т.ч. через Spring Boot стартер), очень многие приключения с реактивщиной начинаются словами: *“Заходит как-то раз WebMVC в classpath к реактивному сервису…”*

В зависимости от того, какие реактивные библиотеки уже используются в приложении, попытка явно или транзитивно втянуть зависимость от WebMVC может закончиться по-разному. Например, Spring Cloud Gateway (построенный на WebFlux) [обозначает](https://docs.spring.io/spring-cloud-gateway/docs/3.0.5/reference/html/#gateway-starter) свою радикальную позицию сразу при запуске приложения:

```text
11:20:19.437  WARN - [           main] sid: rid: - GatewayClassPathWarningAutoConfiguration : 
**********************************************************
Spring MVC found on classpath, which is incompatible with Spring Cloud Gateway at this time.
Please remove spring-boot-starter-web dependency.
**********************************************************

***************************
APPLICATION FAILED TO START
***************************
```

Это хорошо, потому что fail-fast. Однако так везёт далеко не всегда. В более общем (и распространённом) случае, если просто смешать зависимости от WebMVC и WebFlux в одном classpath’е, то смесь не взорвётся, и ни во время компиляции, ни на запуске, скорее всего, ничего не сломается (по крайней мере, со стороны Spring). Даже предупреждений в логе не будет. И это не диверсия, а [сознательное](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/web-reactive.html#webflux) решение разработчиков:

> Applications can use one or the other module or, in some cases, both — for example, Spring MVC controllers with the reactive `WebClient`.

А какой стек тогда поднимется: сервлетный (по умолчанию на Tomcat) или реактивный (по умолчанию на Netty)? Вероятно, авторы руководствовались упомянутым в цитате выше примером, поэтому в таком случае поднимется именно сервлетный стек. Логика выбора прописана в методе `WebApplicationType#deduceFromClasspath`, который вызывается на самом старте Spring Boot приложения:

```java
static WebApplicationType deduceFromClasspath() {
	if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) 
	&& !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
	&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
		return WebApplicationType.REACTIVE;
	}
	for (String className : SERVLET_INDICATOR_CLASSES) {
		if (!ClassUtils.isPresent(className, null)) {
			return WebApplicationType.NONE;
		}
	}
	return WebApplicationType.SERVLET;
}
```

 Как видно, эта логика опирается на состав classpath, а значит, чтобы на неё повлиять, придётся п(р)оиграть с зависимостями.

## Подготовка зависимостей

Есть несколько неоднозначных способов разрулить зависимости приложения так, чтобы снизить вероятность коллизий между реактивными и сервлетными компонентами. Ниже приведены некоторые из этих способов на примере Gradle.

#### “В лоб”

Если WebMVC был втянут транзитивно, и с виду это не жизненно важная зависимость, то её можно просто выкинуть (говорили они):

```groovy
configurations {
  implementation.exclude group: 'org.springframework.boot', module: 'spring-boot-starter-web'
}
```

Однако тогда надо быть готовым к тому, что у этого “недозвездолёта” уже в воздухе (в runtime) начнут отваливаться части. Например так:

```text
***************************
APPLICATION FAILED TO START
***************************
Description:
Parameter 6 of constructor in springfox.documentation.spring.web.plugins.DocumentationPluginsBootstrapper required a bean of type 'javax.servlet.ServletContext' that could not be found.
Action:
Consider defining a bean of type 'javax.servlet.ServletContext' in your configuration.
```

Так происходит потому, что вместе с WebMVC исчезает зависимость от Tomcat или иного сервлет-контейнера, а вместе с ним – и зависимость от Servlet API, что совсем плохо для сторонних библиотек и прикладного кода, повязанного на сервлетном стеке.

По этой причине такой вариант годится далеко не во всех случаях.

#### Выключатель

Иногда поломавшаяся зависимость просто не может жить вне сервлетного стека, поэтому вполне адекватный вариант – отказаться от неё. Например, так [приходится](https://stackoverflow.com/a/51370760) поступить со встраиваемым мониторингом JavaMelody:

```groovy
configurations {
  compileClasspath.exclude group: 'net.bull.javamelody', module: 'javamelody-spring-boot-starter'
  runtime.exclude group: 'net.bull.javamelody', module: 'javamelody-spring-boot-starter'
}
```

Однако бывает так, что скандальный компонент нужен в classpath, но не в Spring-контексте. Тогда может пригодиться вариант следующий.

#### Условность

Если поломавшиеся Spring-компоненты не критичны для реактивного приложения, их можно временно (правда-правда) отключить для реактивного стека путём навешивания вот такой аннотации:

```java
@ConditionalOnWebApplication(type = SERVLET)
public class LogRequestAndResponseAspect {
```

Она позволит бину регистрироваться в контексте, только если приложение сервлетное, не вызывая проблем в реактивном режиме. Понятно, что такой ~~костыль~~ вариант нельзя считать полноценным решением.

#### Компромисс

Как видно, предыдущие варианты сводятся к тому, что мы сознательно отстреливаем себе разные части тела, пытаясь угадать, какие из них не понадобятся в дальнейшем. И хотя совсем без таких решений порой не обойтись, снизить их количество можно за счёт исключения WebMVC, но с оставлением Serlvet API в classpath, например, так:

```groovy
dependencies {
    // ...
    configurations {
        implementation.exclude group: 'org.springframework.boot', module: 'spring-boot-starter-web'
    }
    implementation group: 'javax.servlet', name: 'javax.servlet-api'
    implementation group: 'org.springframework.boot', name: 'spring-boot-starter-webflux'
    // ...
}
```

 Задумка здесь в том, чтобы устранить WebMVC, но не настолько, чтобы в runtime утонуть в бесчисленных `NoClassDefFoundError`. Другими словами, мы не хотим поднимать полноценный сервлетный стек, но хотим, чтобы классы от Serlvet API остались в classpath.

Разумеется, само по себе такое решение не позволит обычному servlet-based коду работать на реактивном стеке. Так, например, любая попытка обращения к текущему HTTP-запросу через метод `RequestContextHolder#currentRequestAttributes` (в том числе изнутри фреймворка) обернётся вот таким взрывом:

> `java.lang.IllegalStateException`: No thread-bound request found: Are you referring to request attributes outside of an actual web request, or processing a request outside of the originally receiving thread? If you are actually operating within a web request and still receive this message, your code is probably running outside of DispatcherServlet: In this case, use RequestContextListener or RequestContextFilter to expose the current request.

Так происходит потому, что к текущему потоку не привязаны данные о выполняемом запросе. Кто и когда их должен был привязать, разберём на следующем прикладном примере.

## Поддержка MDC меток

Одна из самых популярных задач, взрывающих мозг честным людям при переходе с сервлетного стека на реактивный, – это обеспечение повсеместного вывода [MDC-меток](http://slf4j.org/manual.html#mdc) в логах приложения. 

{{< spoiler text="Что такое MDC?" >}}

Это строковые диагностические метки, которые позволяют легко находить в логах записи, относящиеся к одному этапу/аспекту/модулю приложения. Хрестоматийный пример: идентификатор пользовательского запроса (request id, `rid`) – позволяет выявить в логе все записи, относящиеся к обработке каждого входящего запроса, из какой бы части приложения эти записи не были сделаны.

{{< /spoiler >}}

###### Короче (TL;DR)

*Если не хочется/некогда вникать в эту проблему и стоящий за ней сдвиг понимания, можно сразу посмотреть [решение](https://gist.github.com/Toparvion/90ebbdef9d31185a785670d9c2a86762){{< icon name="github" pack="fab" >}}.*

### Проблема

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

Однако с переходом на стек реактивный, вся эта прелесть работать перестала. И дело не только в том, что принцип “один запрос – один поток” перестал соблюдаться, но и в том, что фильтры из Servlet API по понятным причинам вообще перестал как-либо учитываться и всё происходящее в них игнорируется.

### Вывод решения

{{< spoiler text="*Примечание для буквоедов*" >}}

Эрудированный читатель наверняка знает, что задаче проброса MDC-меток в документации на Project Reactor посвящен [отдельный пункт в FAQ](https://projectreactor.io/docs/core/release/reference/#faq.mdc). Почему бы просто не последовать ему? Потому что он предполагает написание **новой** логики в реактивном стиле, в то время как мы решаем задачу адаптации **существующей** (императивной) логики к работе в реактивном окружении. 

{{< /spoiler >}}

#### Переход на WebFilter

На первый взгляд может показаться, что “мигрировать” проброс MDC-меток с сервлетного стека на реактивный очень просто – достаточно поменять приведенный выше фильтр на соответствующий реактивный аналог `org.springframework.web.server.WebFilter`. Его основное отличие состоит лишь в том, что метод `filter` возвращает уже не чистый `void`, а обёртку `Mono<Void>`. Ну, и пара запрос/ответ собрана в один объект `exchange`. В остальном смысл, вроде как, тот же, так что переписать не сложно:

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

#### Прививка реактивности

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

… но только до тех пор, пока вся обработка запроса (и построение конвейера, и его выполнение) остаётся в одном потоке. По умолчанию Reactor не будет плодить потоки или как-либо переключать их без ведома разработчика. Однако одна из его сильнейших сторон как раз и состоит в том, чтобы это делать, причем с минимальным головняком для разработчика. А понадобиться это может, например, для того, чтобы вынести какой-либо старый синхронный (еще пока не реактивный) код в отдельный поток, взятый из специального пула (не конкурирующего с основным). В Project Reactor это [делается](https://projectreactor.io/docs/core/release/reference/#faq.wrap-blocking) буквально одним оператором `subscribeOn(Schedulers.boundedElastic())`, и в рассматриваемом примере приведёт вот к чему:

```text
12:34:08.192 rid:123 TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Выставлена метка rid=123
12:34:08.238 rid:     INFO --- [oundedElastic-1] p.t.s.g.g.GatewayDemoWebFluxApplication  : Синхронно перенаправляю запрос...
12:34:08.332 rid:    TRACE --- [ctor-http-nio-3] p.t.s.g.gatewaydemo.ReactiveMdcFilter    : Убрана метка rid=123
```

Здесь видно, что прикладная логика выполняется в потоке `boundedElastic-1`, отличном от исходного `reactor-http-nio-3`, поэтому поточно-локальная переменная недоступна, и MDC-метка `rid` не выведена. В силу врождённой асинхронности и неблокируемости, Reactor может жонглировать потоками, нарушая некогда непреложный принцип “один запрос – один поток”. Однако это же способность является его сильной стороной, и с ней нужно уметь уживаться.

#### Дополнение

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

#### Двойное гражданство

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

В таком виде у нас появляется “подстилка” под оба режима работы – и сервлетный, и реактивный, но каждая “оживает” только в своём режиме и не мешается в другом. Полная версия класса ==приведена== [здесь](https://gist.github.com/Toparvion/90ebbdef9d31185a785670d9c2a86762){{< icon name="github" pack="fab" >}}.

## Поддержка `RequestContextHolder`

### Проблема

Возможность заполучить текущий обрабатываемый запрос практически из любого места в коде, не пробрасывая его сквозь сигнатуры методов, – бесценная возможность WebMVC. Как и в случае с MDC-метками, она держится на механизме поточно-локальных переменных – в самом классе `RequestContextHolder` есть поле:

```java
private static final ThreadLocal<RequestAttributes> requestAttributesHolder
```

, значение которого можно получить статическим методом `currentRequestAttributes()` (и не только им). А через него уже можно выйти на текущий запрос `HttpServletRequest`. Вообще говоря, полагаться на вызов статического метода из недр приложения – сомнительная затея, ведь она, как минимум, затрудняет покрытие кода модульными тестами (статические методы сложнее `Mock'`ировать). Тем не менее, её часто используют в инфраструктурной логике (например, в аспектах), да и в самом фреймворке она тоже широко используется.

Как не трудно догадаться, с приходом реактивщины этот механизм перестаёт работать. И дело тут не столько в слове `Servlet` в названии класса запроса, сколько в том, что в реактивном веб-приложении нет чёткой связки “один запрос – один поток”. Как следствие, любая попытка вызвать метод `currentRequestAttributes()` из некогда сервлетной логики оканчивается вот этим:

> `java.lang.IllegalStateException`: No thread-bound request found: Are you referring to request attributes outside of an actual web request, or processing a request outside of the originally receiving thread? If you are actually operating within a web request and still receive this message, your code is probably running outside of DispatcherServlet: In this case, use RequestContextListener or RequestContextFilter to expose the current request.

Несмотря на подробную ошибку, ни один из предлагаемых ею классов здесь не поможет, потому что они работают только на сервлетном стеке.

### Решение

В отличие от случая с MDC, здесь не удастся обойтись лишь добавлением какого-то нового класса/метода, который сделает “всё хорошо”. Придётся залезть в ту логику, которая обращается к `RequestContextHolder'`у, и адаптировать её к работе одновременно на двух стеках: сервлетном и реактивном. Крупными мазками эта адаптация состоит в следующем:

1. Избавиться от статической зависимости.
2. Перейти на обобщённый класс запроса (попутно изменив подход к получению его данных).
3. Обеспечить доступность “реактивного” запроса из единой точки обращения.

#### Удаление статической зависимости

Даже безотносительно к решаемой здесь задаче переход от вызова статического метода к вызову instance-метода у Spring-бина – хорошая идея как с точки зрения тестируемости кода (см. описание проблемы), так и с точки зрения наведения порядка, ведь это позволит сделать зависимость от такого метода более явной и управляемой.

В случае с классом `RequestContextHolder` можно начать с простейшего и обернуть его в специально заведённый для этого бин, например, такой:

```java
@Component
public class HttpRequestAccessor {
  public HttpServletRequest fetchCurrentRequest() {             // [1]
    var servletRequestAttributes = (ServletRequestAttributes) 
      RequestContextHolder.currentRequestAttributes();          // [2]
    return servletRequestAttributes.getRequest();
  }
}
```

:one: Метод уже не статический.  
:two: `RequestAttributes` – это абстракция для работы с данными запроса в различных окружениях и на различных областях видимости. Из неё можно получить и сам запрос.

Тогда код, который раньше выглядел примерно как:

```java
((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest()
```

, теперь нужно будет переписать на:

```java
httpRequestAccessor.fetchCurrentRequest()
```

, где `httpRequestAccessor` – экземпляр класса `HttpRequestAccessor`, внедрённый в текущий класс, как правило, через `@Autowired`.

И хотя для модульных тестов это уже большое благо, для адаптации к реактивному стеку сделанная правка – условие необходимое, но не достаточное. Нужно ещё как-то преобразовать тип результата (`HttpServletRequest`), чтобы он не был так сильно привязан к сервлетному стеку.

#### Обобщение класса запроса

Первое, что может прийти в голову, это как-то подобрать/написать такую имплементацию интерфейса `HttpServletRequest`, чтобы она хорошо работала на реактивном стеке, а обращающийся к ней код этого не замечал. Однако помимо эстетического изъяна в виде слова `Servlet` в имени интерфейса, у этого подхода ещё есть сложности в том, что некоторые методы интерфейса (коих ~~прорва~~ весьма немало) имеют довольно сильную сервлетную специфику, а значит, их придётся либо заглушать чем-то вроде `NotImplementedException`, либо городить нетривиальную логику по мимикрированию под реактивный стек. Ни то, ни другое перспективным не выглядит.

К счастью, в пакете `org.springframework.http` есть интерфейс `HttpRequest`, который является достаточно общей абстракцией, чтобы под него подпадали интерфейсы как “сервлетного” запроса, так и “реактивного”.  Правда, каждый со своими особенностями.

Интерфейс **“реактивных”** запросов `org.springframework.http.server.reactive.ServerHttpRequest` как раз является наследником интерфейса `HttpRequest`. Это значит, что в обобщённой логике можно будет проверить экземпляр `HttpRequest` на `instanceof` от `ServerHttpRequest` и таким образом понять, “реактивный” это запрос или нет. Всё так, только с такой проверкой надо быть осторожным – в Spring есть ещё один интерфейс с точно таким же именем `ServerHttpRequest`, но уже в пакете `org.springframework.http.server`. Хуже того, он тоже расширяет интерфейс `HttpRequest`. Вся разница только в том, что (согласно javadoc) этот интерфейс:

> Represents a server-side HTTP request.

, в то время как нужный нам:

> Represents a **reactive** server-side HTTP request.

, т.е. один является абстракцией универсальной, а второй – частной, для реактивного стека. Почему при этом они делят одно имя, для автора остаётся загадкой.

Интерфейсу **“сервлетных”** запросов `HttpServletRequest` такие проблемы чужды – он вообще не является наследником `HttpRequest`. А чтобы позволить ему хоть как-то вступить в этот элитный клуб, разработчики Spring добавили класс `org.springframework.http.server.ServletServerHttpRequest`, который решает эту задачу за счёт того, что:

* является делегатом к заданному экземпляру `HttpServletRequest`;
* имплементирует интерфейс `HttpRequest`, делегируя вызовы его методов оборачиваемому объекту с попутной адаптацией.

Благодаря этому классу, при проверках объектов `HttpRequest` на `instanceof` теперь тоже можно будет отличать “сервлетные” запросы от остальных, но для этого они должны быть предварительно обёрнуты в этот класс.

Вся картина этих семейных отношений через призму IDEA выглядит примерно так:

{{< figure src="img/class-diagram.png" caption="Генеалогическое древо интерфейса HttpRequest" numbered="false" >}}

В целом, уже неплохо, но этого мало, потому что состав методов интерфейса `HttpRequest` (даже включая наследованные) весьма скуден:

{{< figure src="img/http-request-methods.png" caption="Методы интерфейса HttpRequest" numbered="false" >}}

В нём нет методов для получения многих часто используемых данных запроса, например:

* URI-параметров (аналог `javax.servlet.ServletRequest#getParameter`),
* cookies (аналог `javax.servlet.http.HttpServletRequest#getCookies`),
* атрибутов (аналог `javax.servlet.ServletRequest#getAttribute`).

Нет в нём и аналога метода `ServletRequest#getInputStream` для чтения тела запроса. Да и вообще работа с телами запросов/ответов в реактивном стеке существенно отличается от сервлетного и заслуживает отдельной заметки.

А здесь для примера будут рассмотрены только приведенные выше пункты.

Пользуясь выделенным ранее общим интерфейсом `HttpRequest` и осознанием того, что какие-то данные запроса придётся извлекать отдельными методами, можно уже сейчас наметить примерный скелет этих методов. Для простоты положим, что они будут находиться в том же классе `HttpRequestAccessor`, а выглядеть будут примерно так:

```java
public Object getSomethingFrom(HttpRequest request) {
  // если запрос сервлетный
  if (request instanceof ServletServerHttpRequest wrapper) {          // [1]
    HttpServletRequest servletRequest = wrapper.getServletRequest();  // [2]
    // извлекаем что-то из сервлетного запроса
    return something;
  }
  // если запрос реактивный
  if (request instanceof ServerHttpRequest reactiveRequest) {
    // извлекаем что-то из реактивного запроса
    return something;
  }
  // если запрос - неведомая дичь
  throw new IllegalArgumentException("Неизвестный тип запроса: " + request.getClass());
}
```

 :one: Используем [Pattern Matching](https://docs.oracle.com/en/java/javase/16/language/pattern-matching-instanceof-operator.html) (JDK 16+) для  краткости.  
:two: Избавляемся от обёртки.

##### Извлечение URI-параметров

Семейство методов `ServletRequest#getParameter*` в сервлетном стеке умеет работать как с обычными query-параметрами, передаваемыми в URL после знака `?`, так и с параметрами POST-запросов, передаваемыми в теле (например, при сабмите HTML-формы с `Content-Type: application/x-www-form-urlencoded`). В реактивном стеке для этого введён отдельный метод `ServerWebExchange#getFormData`. Очевидно, в таких случаях требуется парсинг тела запроса, но о нём будет рассказано чуть позже. А в этом пункте речь пойдёт только об обычных советских URI-параметрах, используемых чаще всего с HTTP-методом `GET`.

В случае с “**реактивным**” запросом всё предельно просто – есть метод `ServerHttpRequest#getQueryParams`, который возвращает `MultiValueMap<String, String>` – Spring’овый аналог коллекции `Map<String, List<String>>`. Такое нагромождение нужно для того, чтобы учесть случай, когда в запросе присутствует параметр, указанный более одного раза, например:

```http
GET http://localhost:8081/inspect?rid=123&rid=567
```

В этом случае значения `123` и `567` станут элементами списка, доступного по ключу `rid`. Кажется, что это вполне удобный формат для представления результата, поэтому он и будет принят за основной далее.

А у “**сервлетных**” запросов аналогичный метод `ServletRequest#getParameterMap` несколько отличается: он не только умеет возвращать параметры из тела POST-запросов, но и делает это в другом виде – `Map<String, String[]>`, поэтому придётся адаптировать этот вид к принятому выше `MultiValueMap<String, String>`.

С учётом этих договорённостей, метод получения URI-параметров в классе `HttpRequestAccessor` может выглядеть примерно так:

```java
public MultiValueMap<String, String> getParameters(HttpRequest request) {
  // если запрос сервлетный
  if (request instanceof ServletServerHttpRequest wrapper) {
    HttpServletRequest servletRequest = wrapper.getServletRequest();
    Map<String, String[]> parameterMap = servletRequest.getParameterMap();
    Map<String, List<String>> rawMap = parameterMap.entrySet().stream()  // [1]
      .map(entry -> Map.entry(entry.getKey(), Arrays.asList(entry.getValue())))
      .collect(toMap(Map.Entry::getKey, Map.Entry::getValue));
    return toMultiValueMap(rawMap);                                      // [2]
  }
  // если запрос реактивный
  if (request instanceof ServerHttpRequest serverHttpRequest) {
    return serverHttpRequest.getQueryParams();                           // [3]
  }
  // если запрос - неведомая дичь
  throw new IllegalArgumentException("Неизвестный тип запроса: " + request.getClass());
}
```

:one: Превращаем ` Map<String, String[]>` в `Map<String, List<String>>`. Наверняка есть более изящные способы это сделать.  
:two: Адаптируем к `MultiValueMap<String, String>`.  
:three: Вот везде бы так!

##### Извлечение cookies

С печеньками всё несколько проще – в соответствующих методах сервлетного и реактивного стеков нет семантических отличий (насколько известно автору). Однако отличия снова есть в формате возвращаемых результатов, а также в том, какие классы используются для представления cookies. В сервлетном стеке для этого служит `javax.servlet.http.Cookie`, который, очевидно, не стоит отдавать наружу, чтобы не порождать зависимость от Servlet API. В реактивном стеке для того же используется гораздо более общий класс `org.springframework.http.HttpCookie`, не привязанный к конкретному стеку и потому кажущийся более подходящим. Таким образом, получение линейного списка cookies можно оформить примерно в такой метод:

```java
public List<HttpCookie> getCookies(HttpRequest request) {
  // если запрос реактивный
  if (request instanceof ServerHttpRequest reactiveRequest) {
    return reactiveRequest.getCookies()
      .values().stream()
      .flatMap(List::stream)             // [1]
      .toList();
  }
  // если запрос сервлетный
  if (request instanceof ServletServerHttpRequest servletRequest) {
    Cookie[] cookies = servletRequest.getServletRequest().getCookies();
    if (cookies == null) {
      return List.of();
    }
    return Arrays.stream(cookies)
      .map(servletCookie ->              // [2]
           new HttpCookie(servletCookie.getName(), servletCookie.getValue()))
      .toList();
  }
  // если запрос - неведомая дичь
  throw new IllegalArgumentException("Неизвестный тип запроса: " + request.getClass());
}
```

  :one: Для простоты дальнейшей демонстрации превращаем `MultiValueMap<String, HttpCookie>` в линейный список `List<HttpCookie>` (хотя в production-коде это было бы скорее вредительством).  
:two: Превращаем “сервлетные” cookies в абстрактные.

##### Извлечение атрибутов

В отличие от параметров, заголовков, cookies и тела, атрибуты не являются частью протокола HTTP. Это своего рода внутренний механизм серверных фреймворков для передачи мета-данных между этапами обработки запроса внутри сервера. Соответственно, и имплементация этого механизма может отличаться от фреймворка к фреймворку. Так, например, в Spring WebFlux нет понятия “атрибуты **запроса**”, вместо этого там используются атрибуты **обмена** – сущности, объединяющей запрос и ответ в рамках одного такта взаимодействия между клиентом и сервером. Обмен представлен интерфейсом `org.springframework.web.server.ServerWebExchange`, у которого есть методы `getAttribute*()`. А в Servlet API для той же цели служат методы `javax.servlet.ServletRequest#getAttribute*()`. В обоих случаях тип значения атрибутов объявлен как `Object`, чтобы через них можно было передавать любые объекты, а не только строки. Это вполне допустимо благодаря тому, что наружу (клиентам) атрибуты, как правило, не отдаются.

Поскольку атрибуты могут использоваться как фреймворками, так и прикладной логикой, во избежание коллизий имена атрибутов должны следовать тем же правилам, что и имена Java-пакетов, в частности (выдержка из javadoc на `ServletRequest`):

> Names beginning with `java.*` and `javax.*` are reserved for use by the Servlet specification. Names beginning with `sun.*, com.sun.*, oracle.*` and `com.oracle.*`) are reserved for use by Oracle Corporation.

Этого придерживается и Spring – он предваряет имена своих вспомогательных атрибутов (например, `bestMatchingHandler` и `producibleMediaTypes`) полным именем класса `HandlerMapping`. И хотя имена и смысл самих атрибутов в Spring WebMVC и WebFlux почти полностью совпадают, фактические имена атрибутов **различаются**, потому что в каждом фреймворке есть **свой** класс `HandlerMapping`:

```text
org.springframework.web.servlet.HandlerMapping
org.springframework.web.reactive.HandlerMapping
```

Из этого следует, что просто извлекать атрибуты из разных мест для WebMVC и WebFlux недостаточно, нужно ещё учитывать их имена. Однако здесь важно не увлечься и не начать писать логику преобразования имён одного фреймворка в имена другого. Ведь, если задуматься, атрибуты каждого фреймворка должны запрашиваться только в рамках работы на его родном стеке. Другими словами, попытка запросить атрибут, например, с *сервлетным* именем при работе на *реактивном* стеке – это явный признак произошедшей путаницы. И в таком случае нужна не “умная” логика преобразования, а тревога, чтобы как можно раньше “подсветить” место потенциального источника проблем. Для сервлетного стека первоклассный программист (обучающийся в первом классе) написал бы эту подсветку примерно так:

```java
Assert.doesNotContain(attributeName, "org.springframework.web.reactive.HandlerMapping", 
  "В сервлетном режиме не должен запрашиваться реактивный атрибут " + attributeName);
```

, где `Assert` – это `org.springframework.util.Assert`.

Придерживаясь взятого ранее курса на возврат *полной* коллекции запрашиваемых данных запроса, класс `HttpRequestAccessor` мог бы предоставлять для атрибутов вот такой метод:

```java
public Map<String, String> getAttributes(HttpRequest httpRequest) {
  // если запрос сервлетный
  if (httpRequest instanceof ServletServerHttpRequest wrapper) {
    HttpServletRequest httpServletRequest = wrapper.getServletRequest();
    Map<String, String> attributes = new LinkedHashMap<>();
    Enumeration<String> attributeNames = httpServletRequest.getAttributeNames();
    while (attributeNames.hasMoreElements()) {                           // [1]
      String attributeName = attributeNames.nextElement();
      Object attributeValue = httpServletRequest.getAttribute(attributeName);
      attributes.put(attributeName, String.valueOf(attributeValue));     // [2]
    }
    return attributes;
  }
  // если запрос реактивный
  if (httpRequest instanceof ServerHttpRequest) {
    ServerWebExchange currentExchange = REACTIVE_EXCHANGE_HOLDER.get();  // [3]
    return currentExchange.getAttributes()
      .entrySet().stream()
      .map(entry -> Map.entry(entry.getKey(), String.valueOf(entry.getValue())))
      .collect(toMap(Map.Entry::getKey, Map.Entry::getValue));
  }
  // если запрос - неведомая дичь
  throw new IllegalArgumentException("Неизвестный класс запроса: " + httpRequest.getClass());
}
```

:one: Атрибуты в Servlet API нельзя получить сразу все, поэтому приходится перебирать их по именам.  
:two: Для наглядности сводим значение атрибута к строке (в production-коде делать так не надо).  
:three: Что такое `REACTIVE_EXCHANGE_HOLDER` – см. ниже.

#### Обеспечение доступности запроса





---

## Поддержка `@Around`-аспектов

### Проблема

Ещё одна любопытная задачка, возникающая при внедрении реактивного стека в сервлетные приложения – это поддержка AOP-аспектов, оборачивающих тело целевого метода. Такие аспекты декларируются с аннотацией `org.aspectj.lang.annotation.Around` и имеют возможность повлиять как на аргументы перехватываемого метода, так и на его результат, вплоть до полной его замены. Например, вот так может выглядеть примитивный аспект, логирующий аргументы и результат любого метода, помеченного аннотацией `@GetMapping`:

```java
@Aspect
@Component
public class LoggingAspect {
  private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);
  
  @Around("within(pro.toparvion.sample.gateway.gatewaydemo..*) " +    // [1]
      " && execution(@org.springframework.web.bind.annotation.GetMapping * *.*(..))") // [2]
  public Object logArgsAndResult(ProceedingJoinPoint pjp) throws Throwable {
    Object[] args = pjp.getArgs();             // [3]
    String targetMethodName = pjp.getSignature().getName();
    log.debug("Метод '{}' вызван с аргументами: {}", targetMethodName, args);
    Object response = pjp.proceed(args);       // [4]
    log.debug("Метод '{}' ответил: {}", targetMethodName, response);
    return response;
  }
}
```

:one: Ограничиваем область действия аспекта, чтобы не [напороться](https://twitter.com/toparvion/status/1177542707541086208?s=20).  
:two: Задаём шаблон методов, на которые должен действовать аспект.  
:three: Получаем аргументы и имя целевого метода из точки вкрапления.  
:four: После логирования аргументов делегируем управление целевому методу, чтобы потом сразу залогировать его результат. Для простоты аспект не учитывает случай, когда метод завершается исключением.

 ***Проблема*** здесь, в сущности, та же, что и с фильтром для поддержки MDC: из-за “двухэтапного” исполнения реактивного конвейера аспект захватит лишь результат *декларации* конвейера, но не реально курсирующие по нему данные:

```text
16:24:18.136 DEBUG --- [ctor-http-nio-3] p.t.s.gateway.gatewaydemo.LoggingAspect  : Метод 'proxy' ответил: MonoPeekTerminal
```

, т.е. вместо действительного результата в логах всегда будет числиться `MonoPeekTerminal` (для данного примера).

### Решение

Решением может стать такое же разделение аспекта на два: для сервлетного и реактивного режимов (при помощи аннотации `@ConditionalOnWebApplication`). Однако, в отличие от случая с MDC, здесь мы не привязаны к API веб-фильтров, а значит, можем переписать аспект так, чтобы он один умел работать в обоих режимах. Это может быть особенно ценным, если логика аспекта достаточно сложна и/или объёмна, чтобы сохранить её как можно более [DRY](https://ru.wikipedia.org/wiki/Don%E2%80%99t_repeat_yourself).

Чтобы это сделать, нужно:

1. Уметь отличать реактивные методы от императивных;
2. Уметь назначать желаемое поведение *отложенно*, т.е. не в момент применения аспекта, а позже, когда по конвейеру будут проходить данные.

Первое достаточно легко сделать анализом типа результата метода: если он представляет собой реактивный “хвостик” в виде `Mono<?>` или `Flux<?>`, то и весь метод можно считать реактивным. Второе решается использованием [соответствующих](https://projectreactor.io/docs/core/release/reference/#which.peeking) реактивных операторов: в рассматриваемом простом случае для `Mono` это будет метод `doOnSuccess()`, а для `Flux` – `doOnNext()`. Кроме того, чтобы не повторять само логирование в каждом месте, вызов `log.debug("Метод '{}' ответил: {}..."` стоит оформить в метод `logResult()`. Тогда основной метод аспекта (точнее, его окончание) преобразится вот таким образом:

```java
  @Around("within(pro.toparvion.sample.gateway.gatewaydemo..*) " +
      " && execution(@org.springframework.web.bind.annotation.GetMapping * *.*(..))")
  public Object logArgsAndResult(ProceedingJoinPoint pjp) throws Throwable {
    Object[] args = pjp.getArgs();
    String targetMethodName = pjp.getSignature().getName();
    log.debug("Метод '{}' вызван с аргументами: {}", targetMethodName, args);
    Object resultTail = pjp.proceed(args);
    return assignResultLoggingBehavior(targetMethodName, resultTail);   // [1]
  }
```

:one: Теперь здесь нет явного логирования результата. Вместо него вызывается некое назначение (декорирование) ответа, которое на самом деле может сводиться и к непосредственному логированию. Это видно в коде нового метода `assignResultLoggingBehavior`:

```java
  private Object assignResultLoggingBehavior(String targetMethodName, Object tail) {
    if (tail instanceof Mono<?> monoResult) {         // [1]
      tail = monoResult.doOnSuccess(result -> logResult(targetMethodName, result));
    } else if (tail instanceof Flux<?> fluxResult) {  // [2]
      tail = fluxResult.doOnNext(result -> logResult(targetMethodName, result));
    } else {
      logResult(targetMethodName, tail);              // [3]
    }
    return tail;                                      // [4]
  }
```

:one: Используем [Pattern Matching](https://docs.oracle.com/en/java/javase/16/language/pattern-matching-instanceof-operator.html) (JDK 16+) для сокращения записи.  
:two: В случае с HTTP-вызовом результат в виде `Flux` соответствует стриминговому ответу сервера, например, [SSE](https://ru.wikipedia.org/wiki/Server-sent_events) (просто FYI).  
:three: Если результат не реактивный, просто логируем результат сразу.  
:four: Возвращаем результат, чтобы на него можно было подписаться, либо сразу вернуть клиенту. 

В таком виде аспект сможет воевать на два фронта, например, будучи оформленным в инфраструктурную библиотеку, его можно будет использовать как с привычными сервлетными приложениями, так и с новомодными реактивными. Однако важно помнить, что в `import`'ах у него есть классы из Project Reactor, а значит, при использовании этого класса в runtimeClasspath **обязана присутствовать библиотека** `io.projectreactor:reactor-core`, даже если аспект применяется к сервлетному приложению.

Разумеется, такой аспект далёк от состояния production-ready, хотя бы потому, что:

* не учитывает, что аргументы тоже могут быть реактивными;
* не обрабатывает исключения, летящие из жерла целевого метода;
* декларирован как `@Around`, хотя далеко не всегда требуется именно двусторонняя обёртка (часто достаточно обойтись аннотациями `@Before` или `@After`).

Однако в качестве отправной точки его должно быть достаточно. Полный исходный код аспекта можно посмотреть ==здесь==.

## Поддержка Spring Cloud OpenFeign

В отличие от предыдущих пунктов, рассматриваемые здесь две проблемы нельзя назвать широко распространёнными. Однако тем “счастливчикам”, кто всё же сталкивается с ними, от того не легче.

### Проблема 1

Она [констатирована](https://docs.spring.io/spring-cloud-openfeign/docs/3.0.5/reference/html/#reactive-support) прямо в документации на Spring Cloud OpenFeign:

> As the [OpenFeign project](https://github.com/OpenFeign/feign) does not currently support reactive clients, such as [Spring WebClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html), neither does Spring Cloud OpenFeign.

Другими словами, если в каком-то проекте используется высокоуровневый декларативный HTTP-клиент на основе библиотеки OpenFeign, то под его капотом не сможет работать соответствующий низкоуровневый реактивный HTTP-клиент. Это относится к Spring Cloud OpenFeign версии 3.0.5 и остаётся актуальным, как минимум, на 22.11.2021. 

Сразу за этой констатацией идёт многообещающее:

> We will add support for it here as soon as it becomes available in the core project.

, и в самом OpenFeign об этом действительно много [разговоров](https://github.com/OpenFeign/feign/issues/361), однако воз и ныне там. А пока всё так, разработчики Spring Cloud предлагают:

> Until that is done, we recommend using [feign-reactive](https://github.com/Playtika/feign-reactive) for Spring WebClient support.

Но этот путь годится только новым проектам, ведь он требует переработки интерфейсов декларативных клиентов – их результаты должны получить обёртки в виде `Mono` или `Flux`. А в текущей задаче мы рассматриваем вариант только с сохранением императивного стиля.

Чем чревата эта проблема? Тем, что в реактивном приложении ненароком вызванный OpenFeign-клиент может заблокировать какой-либо из потоков-обработчиков в Reactor’е и этим парализовать его частично или даже полностью. Коварство проблемы в том, что пока потоков-обработчиков достаточно много, а ответы OpenFeign-клиентам приходят достаточно быстро, деградация, скорее всего, останется незамеченной. По законам жанра это будет происходить в тестовом окружении. И только под серьезной нагрузкой (например, на production, почему бы нет) она начнёт периодически постреливать то там, то там, или вовсе поставит всё колом.

### Решение 1

Если потенциальное место вызова OpenFeign-клиента известно, то его нужно локализовать относительно реактивного конвейера и приправить вызов оператором `.subscribeOn(Schedulers.boundedElastic())` или `.publishOn(Schedulers.boundedElastic())`. Это позволит Reactor’у аллоцировать под вызов отдельный поток, который будет [не жалко](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html#boundedElastic--) на какое-то время заблокировать.

Если же такое место не известно или есть опасение, что оно не одно, то стоит потестировать приложение под наблюдением [BlockHound](https://github.com/reactor/BlockHound) – специального Java-агента, умеющего детектировать блокирующие вызовы из неблокирующих потоков. С его помощью должно получиться выявить проблемные вызовы и либо обернуть их в эластичный ~~бинт~~ пул, либо исключить.

### Проблема 2

Куда “веселее” становится, когда в дело вступает балансировщик нагрузки `spring-cloud-loadbalancer`, который для OpenFeign-клиентов поставляет класс `org.springframework.cloud.loadbalancer.blocking.client.BlockingLoadBalancerClient`. Как не трудно догадаться по его имени, с реактивщиной этот класс дружит плоховато. Из-за этого получается, что с переходом на реактивный стек некогда безупречные OpenFeign-клиенты уже в runtime начинают падать *на каждом* запросе с неприглядным:

```text
java.lang.IllegalStateException: block()/blockFirst()/blockLast() are blocking, which is not supported in thread reactor-http-nio-3
	at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:83) ~[reactor-core-3.4.11.jar:3.4.11]
```

Так происходит потому, что в методе `choose` класса `BlockingLoadBalancerClient` (для балансировщика версии 3.0.4) есть вот такая строчка:

```java
Response<ServiceInstance> loadBalancerResponse = Mono.from(loadBalancer.choose(request)).block();
```

Своим вызовом `block()` она раздражает в методе `reactor.core.publisher.BlockingSingleSubscriber#blockingGet()` проверку:

```java
if (Schedulers.isInNonBlockingThread()) {
  throw new IllegalStateException("block()/blockFirst()/blockLast() are blocking, which is not supported in thread " + Thread.currentThread().getName());
}
```

, которая и роняет выполнение запроса. Любопытно, что под вызовом `isInNonBlockingThread()` скрывается не какая-нибудь реактивная магия, а всего лишь проверка на `Thread.currentThread() instanceof NonBlocking` (наличие специального маркерного интерфейса).

Выброс такого исключения – сознательная мера Project Reactor’а для того, чтобы выявлять и предотвращать выполнение блокирующего кода в потоках-обработчиках, коих всегда немного (как правило, по числу ядер процессора) и которые должны освобождаться максимально быстро (чтобы подхватить другие задачи). В этом заключается часть смысла слова “неблокирующий” в дефиниции любого реактивного фреймворка. И да, это тоже проявление fail-fast, благодаря чему проблемные места можно замечать без применения спецсредств наподобие BlockHound.

### Решение 2

Но что делать с этим нам, прикладным разработчикам? Первое решение – такое же, как для проблемы 1 – [оформлять](https://projectreactor.io/docs/core/release/reference/#faq.wrap-blocking) подобные вызовы в пул потоков, допускающих блокировку, например, `Schedulers.boundedElastic()`.

Но, увы, иногда так не получается. Например, если в рамках плавной миграции на реактивный стек обнаруживается инфраструктурный компонент (например, аспект), использующий OpenFeign-клиента, и этот компонент нельзя менять, потому что он обязан остаться рабочим в сервлетном окружении. Типичная ситуация для рассматриваемой задачи.

В таком случае может (но не обязано) пригодиться решение второе, далеко не самое чисто и надёжное. В Spring Cloud клиент к балансировщику нагрузки поставляется методом `BlockingLoadBalancerClientAutoConfiguration#blockingLoadBalancerClient`, имеющим на себе аннотацию `@ConditionalOnMissingBean`. Значит, проблемный библиотечный бин можно перекрыть своим. И хотя каноны ООП завещают предпочитать наследованию делегирование, здесь для простоты и краткости приведён грязный хак с переопределением одного метода в наследнике класса `BlockingLoadBalancerClient`:

```java
  @Bean
  public LoadBalancerClient blockingLoadBalancerClient(                // [1]
    LoadBalancerClientFactory loadBalancerClientFactory, LoadBalancerProperties properties) {
    return new BlockingLoadBalancerClient(loadBalancerClientFactory, properties) {
      @Override
      public <T> ServiceInstance choose(String serviceId, Request<T> request) {
        ReactiveLoadBalancer<ServiceInstance> loadBalancer = loadBalancerClientFactory.getInstance(serviceId);
        if (loadBalancer == null) {
          return null;
        }
        Publisher<Response<ServiceInstance>> serverInstancePub = loadBalancer.choose(request);
        ServiceInstance[] serviceInstance = new ServiceInstance[1];    // [2]
        Mono.from(serverInstancePub)
          .subscribe(                                                  // [3]
            serviceInstanceResponse -> serviceInstance[0] = serviceInstanceResponse.getServer(),
            oops -> log.error("Failed to choose an instance", oops));  // [4]
        return serviceInstance[0];
      }
    };
  }
```

 :one: Имя метода должно быть именно таким, чтобы перекрыть одноимённый бин из Spring Cloud.  
:two: Такая конструкция нужна в Java для замыкания нефинальных значений в лямбда-выражениях.  
:three: В Project Reactor по умолчанию подписка происходит в том же потоке, что и декларация реактивного конвейера. Пользуясь этим, мы подписываемся на источник инстансов и тут же получаем от него результат выбора, не вызывая ни один из явно блокирующих методов.  
:four: Логируем ошибку, что случайно не поглотить её при возникновении.

{{% callout warning %}} Важно понимать, что такое решение позволяет обмануть не столько Reactor, сколько самого себя, потому что в случае долгого ответа от балансировщика оно не спасёт жизненно важные для Reactor’а потоки от зависания. {{% /callout %}}

Это решение годится лишь на крайний случай и при условии понимания того, как ведёт себя балансировщик для данного приложения.

