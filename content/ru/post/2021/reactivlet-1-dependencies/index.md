---
title: "От сервлетов к реактору 1"
subtitle: "Часть 1. Введение и подготовка зависимостей"
summary: "Часть 1. Введение и подготовка зависимостей"
authors:
  - toparvion
tags:
  - spring
  - reactor
  - servlet
  - webmvc
  - webflux
  - gradle
categories:
  - Performance
series:
  - reactivlet
date: 2021-12-07T08:31:05+07:00
lastmod: 2021-12-07T07:32:05+07:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "[Реакторный зал Ленинградской АЭС](#о-картинке)"
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

Когда рассказывают о прелестях реактивного фреймворка [Spring WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux) и его подкапотном [Project Reactor](https://projectreactor.io/), для примера чаще всего показывают новые, создаваемые с нуля приложения. Однако на практике приходится строить из готовых блоков, в том числе собственных прикладных и инфраструктурных модулей, которые уже написаны в императивном стиле и опираются на сервлетный стек. Как правило, такие модули нельзя/некогда/не охота *(нужное подчеркнуть)* переписывать, поэтому надо как-то адаптировать их к реактивному миру с минимумом правок (а лучше без них вовсе). О некоторых подходах к такой задаче и пойдёт речь в этой серии заметок.

## Вводная

###### Вместо disclaimer’а

*Всё сказанное здесь опирается лишь на небольшой опыт автора в решении одной практической задачи, поэтому статья не претендует на полноту охвата темы ни вширь, ни вглубь. Её цель – лишь обозначить **некоторые** проблемы и **возможные** решения для них.*

Дальнейший материал предполагает хотя бы шапочное знакомство читателя с реактивным стеком технологий в экосистеме Spring, а также наличие практического опыта работы с веб-приложениями на сервлет-контейнере типа Tomcat. Недостающие знания можно почерпнуть, например, в документации на Spring [здесь](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/web-reactive.html#spring-webflux) и [здесь](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/web.html#spring-web).

### Контекст проблемы

{{% callout info %}}

**Историческая справка**

Появившийся в Spring 5 реактивный веб-фреймворк WebFlux был [задуман](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/web-reactive.html#webflux) как невытесняющая альтернатива старому доброму [WebMVC](https://docs.spring.io/spring-framework/docs/5.3.13/reference/html/web.html#spring-web), т.е. существуют и развиваются они оба, но в качестве серверного стека можно использовать только один. Дело не столько в вездесущих обёртках `Mono<>` и `Flux<>` вокруг результатов чуть ли не всех методов реактивного фреймворка, сколько в принципиально иной модели многопоточности под его капотом.

 {{% /callout %}}

Поскольку большинство сервлетных Spring-приложений построено на WebMVC (в том числе через его Spring Boot стартер), очень многие приключения с реактивщиной начинаются словами: *“Заходит как-то раз WebMVC в classpath к реактивному сервису…”*

В зависимости от того, какие реактивные библиотеки используются (или будут) в приложении, их прямая или транзитивная встреча с WebMVC может закончиться по-разному. Например, Spring Cloud Gateway, построенный на WebFlux, [обозначает](https://docs.spring.io/spring-cloud-gateway/docs/3.0.5/reference/html/#gateway-starter) свою радикальную позицию сразу при запуске приложения:

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

А какой же стек тогда поднимется: сервлетный (по умолчанию на Tomcat) или реактивный (по умолчанию на Netty)? Вероятно, авторы руководствовались упомянутым в цитате выше примером, поэтому в таком случае поднимется именно *сервлетный* стек. Логика выбора прописана в методе `WebApplicationType#deduceFromClasspath`, который вызывается на самом старте Spring Boot приложения:

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

#### "Выключатель"

Иногда поломавшаяся зависимость просто не может жить вне сервлетного стека, поэтому вполне адекватный вариант – отказаться от неё. Например, так [приходится](https://stackoverflow.com/a/51370760) поступить со встраиваемым мониторингом JavaMelody:

```groovy
configurations {
  compileClasspath.exclude group: 'net.bull.javamelody', module: 'javamelody-spring-boot-starter'
  runtime.exclude group: 'net.bull.javamelody', module: 'javamelody-spring-boot-starter'
}
```

Однако бывает так, что скандальный компонент нужен в classpath, но не в Spring-контексте. Тогда может пригодиться вариант следующий.

#### "Условность"

Если поломавшиеся Spring-компоненты не критичны при работе на реактивном стеке, их можно условно отключить путём навешивания вот такой аннотации:

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

Разумеется, сам по себе последний вариант не позволит традиционному “сервлетному” коду работать на реактивном стеке. Но зато он наставит на истинный путь упомянутый выше метод `WebApplicationType#deduceFromClasspath`. Останется только починить всё то, что поотваливается ~~на фиг~~ от такого переключения.

Относительно простой вариант распределения зависимостей по Gradle-модулям реализован в прилагаемом [демо-проекте](https://github.com/Toparvion/reactivlet-sample) {{< icon name="github" pack="fab" >}}.

## Попутное резюме

Эта первая заметка отведена под самый неоднозначный этап, который наверняка у каждого читателя проходит как-то по своему. Тем не менее, от его прохождения будет зависеть успех на следующих шагах. А [ближайший ]({{< relref "/post/2021/reactivlet-2-mdc" >}}) из них будет посвящён логированию.

&nbsp;

---

###### О картинке
<sup>
<a href="https://commons.wikimedia.org/wiki/File:RIAN_archive_305017_Leningrad_nuclear_power_plant.jpg">RIA Novosti archive, image #305017 / Alexey Danichev / CC-BY-SA 3.0</a>, <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0</a>, через Викисклад
</sup>
