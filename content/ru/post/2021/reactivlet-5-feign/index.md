---
title: "От сервлетов к реактору 5"
subtitle: "Часть 5. Ремонт OpenFeign-клиентов и заключение"
summary: "Часть 5. Ремонт OpenFeign-клиентов и заключение"
authors:
  - toparvion
tags:
  - spring
  - reactor
  - servlet
  - webmvc
  - webflux
  - cloud
  - openfeign
categories:
  - Performance
series:
  - reactivlet
date: 2021-12-11T08:31:05+07:00
lastmod: 2021-12-11T07:32:05+07:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "[Реакторный зал Обнинской АЭС](#о-картинке)"
  focal_point: ""
  preview_only: false

links:
  - icon: book
    icon_pack: fas
    name: Серия статей ReactivLet
    url: /series/reactivlet/

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

{{< toc >}}


В отличие от предыдущих пунктов, рассматриваемые здесь две проблемы нельзя назвать широко распространёнными. Однако тем “счастливчикам”, кто всё же сталкивается с ними, оттого не легче.

## Проблема 1

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

## Проблема 2

Куда “веселее” становится, когда в дело вступает балансировщик нагрузки `spring-cloud-loadbalancer`, который для OpenFeign-клиентов поставляет класс:

```java
org.springframework.cloud.loadbalancer.blocking.client.BlockingLoadBalancerClient
```

Как не трудно догадаться по его имени, с реактивщиной этот класс дружит плоховато. Из-за этого получается, что с переходом на реактивный стек некогда безупречные OpenFeign-клиенты уже в runtime начинают падать *на каждом* запросе с неприглядным:

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

Но, увы, иногда так не получается. Например, если в рамках плавной миграции на реактивный стек обнаруживается инфраструктурный компонент (например, аспект), использующий OpenFeign-клиента, и этот компонент нельзя менять, потому что он обязан остаться рабочим в сервлетном окружении. Любые совпадения с реальными событиями случайны.

В таком случае может (но не обязано) пригодиться решение второе, далеко не самое чисто и надёжное. В Spring Cloud клиент к балансировщику нагрузки поставляется методом:

```java
BlockingLoadBalancerClientAutoConfiguration#blockingLoadBalancerClient
```

, имеющим на себе аннотацию `@ConditionalOnMissingBean`. Значит, проблемный библиотечный бин можно перекрыть своим. И хотя каноны ООП завещают нам предпочитать наследованию делегирование, здесь для простоты и краткости приведён грязный хак с переопределением одного метода в наследнике класса `BlockingLoadBalancerClient`:

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

{{% callout warning %}} Важно понимать, что такое решение позволяет обмануть не столько Reactor, сколько самого себя, потому что в случае долгого ответа от балансировщика оно не убережёт жизненно важные для Reactor’а потоки от зависания. {{% /callout %}}

Это решение годится лишь на крайний случай и при условии понимания того, как ведёт себя балансировщик для данного приложения.

## Общее заключение

В этой пространной заметке были рассмотрены несколько задач, с которыми может столкнуться прикладной разработчик при попытках подружить императивный код на сервлетном фреймворке Spring WebMVC с кодом на реактивном фреймворке Spring WebFlux:

* Как разрулить зависимости и составить правильный classpath?
* Как поддержать вывод MDC-меток в логах на реактивном стеке?
* Как сохранить доступность текущего запроса из любого места?
* Как сделать аспекты-обёртки терпимыми к реактивному стеку?
* Как сохранить работоспособность HTTP-клиентов на OpenFeign?

Ни весь этот список целиком, ни отдельные его пункты не претендуют на полноту раскрытия темы, поскольку класс таких задач весьма обширен. Однако приведенный материал должен помочь читателю ухватить основные идеи и принципы, чтобы решать подобные задачи гораздо быстрее, чем довелось автору этих строк.

Насколько это (не)удалось, можно и нужно рассказывать в комментариях под этим текстом. Там же приветствуются ссылки на схожие источники информации по этой же теме.

&nbsp;

---

###### О картинке
<sup>
[© РИА Новости / Вячеслав Рунов](https://ria.ru/docs/about/copyright.html) / [Перейти в фотобанк](http://visualrian.ru/images/item/455253)
</sup>
