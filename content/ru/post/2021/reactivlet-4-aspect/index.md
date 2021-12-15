---
title: "От сервлетов к реактору 4"
subtitle: "Часть 4. Унификация `@Around`-аспектов"
summary: "Часть 4. Унификация `@Around`-аспектов"
authors:
  - toparvion
tags:
  - spring
  - reactor
  - servlet
  - webmvc
  - webflux
  - aop
  - aspectj
categories:
  - Performance
series:
  - reactivlet
date: 2021-12-10T08:31:05+07:00
lastmod: 2021-12-10T07:32:05+07:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "[Реакторный зал Нововоронежской АЭС](#о-картинке)"
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

## Проблема

 Проблема здесь, в сущности, та же, что и с фильтром для поддержки MDC: из-за “двухэтапного” исполнения реактивного конвейера аспект захватит лишь результат *декларации* конвейера, но не реально курсирующие по нему данные:

```text
16:24:18.136 DEBUG [ctor-http-nio-3] LoggingAspect : Метод 'proxy' ответил: MonoPeekTerminal
```

, т.е. вместо действительного результата в логах всегда будет числиться `MonoPeekTerminal` (для данного примера).

## Решение

Решением может стать такое же (как с MDC) разделение аспекта на два: для сервлетного и реактивного режимов (при помощи аннотации `@ConditionalOnWebApplication`). Однако, в отличие от случая с MDC, здесь мы не привязаны к API веб-фильтров, а значит, можем переписать аспект так, чтобы он один умел работать в обоих режимах. Это может быть особенно ценным, если логика аспекта достаточно сложна и/или объёмна, чтобы сохранить её как можно более [DRY](https://ru.wikipedia.org/wiki/Don%E2%80%99t_repeat_yourself).

Чтобы это сделать, нужно:

1. Уметь отличать реактивные методы от императивных;
2. Уметь назначать желаемое поведение *отложенно*, т.е. не в момент применения аспекта, а позже, когда по конвейеру будут проходить данные.

Первое достаточно легко сделать анализом типа результата метода: если он представляет собой реактивный “хвостик” в виде `Mono<?>` или `Flux<?>`, то и весь метод можно считать реактивным. Второе решается использованием [соответствующих](https://projectreactor.io/docs/core/release/reference/#which.peeking) реактивных операторов: в рассматриваемом простом случае для `Mono` это будет метод `doOnSuccess()`, а для `Flux` – `doOnNext()`. Кроме того, чтобы не повторять само логирование ответа в каждом месте, вызов `log.debug(...)` стóит оформить в метод `logResult()`. Тогда основной метод аспекта (точнее, его окончание) преобразится вот таким образом:

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

В таком виде аспект сможет воевать на два фронта: например, будучи оформленным в инфраструктурную библиотеку, его можно будет использовать как с привычными сервлетными приложениями, так и с новомодными реактивными. Однако важно помнить, что в `import`'ах у него есть классы из Project Reactor, а значит, при использовании этого класса в runtimeClasspath **должна** присутствовать библиотека `io.projectreactor:reactor-core`, даже если аспект применяется к сервлетному приложению.

Разумеется, такой аспект далёк от состояния production-ready, хотя бы потому, что:

* не учитывает, что аргументы тоже могут быть реактивными;
* не обрабатывает исключения, летящие из жерла целевого метода;
* декларирован как `@Around`, хотя далеко не всегда требуется именно двусторонняя обёртка (часто достаточно обойтись аннотациями `@Before` или `@After`).

Однако в качестве отправной точки его должно быть достаточно.

Полный код аспекта можно найти в прилагаемом [демо-проекте](https://github.com/Toparvion/reactivlet-sample/blob/main/shared/src/main/java/pro/toparvion/sample/reactivlet/shared/LoggingAspect.java){{< icon name="github" pack="fab" >}}.

## Попутное резюме

Эта часть была посвящена одну из самых “магических” механизмов вкрапления в поведение приложений. Теперь и он сможет работать на любом из двух стеков. В [следующей части]({{< relref "/post/2021/reactivlet-5-feign" >}}) речь пойдёт о гораздо более явном инструменте – декларативных HTTP-клиентах.

&nbsp;

---

###### О картинке
<sup>
<a href="https://commons.wikimedia.org/wiki/File:RIAN_archive_342604_The_Novovoronezh_nuclear_power_plant.jpg">RIA Novosti archive, image #342604 / Sergey Pyatakov / CC-BY-SA 3.0</a>, <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0</a>, через Викисклад
</sup>
