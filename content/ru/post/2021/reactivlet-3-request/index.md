---
title: "От сервлетов к реактору 3"
subtitle: "Часть 3. Получение текущего запроса"
summary: "Часть 3. Получение текущего запроса"
authors:
  - toparvion
tags:
  - spring
  - reactor
  - servlet
  - webmvc
  - webflux
  - http
  - cookie
categories:
  - Performance
series:
  - reactivlet
date: 2021-12-09T08:31:05+07:00
lastmod: 2021-12-09T07:32:05+07:00
featured: true
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "[Реакторный зал Курской АЭС](#о-картинке)"
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

Возможность заполучить текущий обрабатываемый запрос практически из любого места в коде (не пробрасывая его сквозь сигнатуры методов) – бесценная фича WebMVC. Как и в [случае]({{< relref "/post/2021/reactivlet-2-mdc" >}}) с MDC-метками, она держится на механизме поточно-локальных переменных – в классе `RequestContextHolder` есть поле:

```java
private static final ThreadLocal<RequestAttributes> requestAttributesHolder
```

, значение которого можно откуда угодно получить публичным статическим методом `currentRequestAttributes()` (и не только им). А через него уже можно выйти на текущий запрос `HttpServletRequest`.

{{< spoiler text="Минутка занудства" >}}

Вообще говоря, полагаться на вызов статического метода из недр приложения – сомнительная затея, ведь она, как минимум, затрудняет покрытие кода модульными тестами (статические методы сложнее `Mock`'ировать). Тем не менее, её часто используют в инфраструктурной логике (например, в аспектах), да и в самом фреймворке она тоже широко используется.

{{< /spoiler >}}

## Проблема

Как не трудно догадаться, с приходом реактивщины этот механизм перестаёт работать. И дело тут не столько в слове `Servlet` в названии класса запроса, сколько в том, что в реактивном веб-приложении нет чёткой связки “один запрос – один поток”. Как следствие, никто по умолчанию не наполняет поточно-локальную переменную, и любая попытка вызвать метод `currentRequestAttributes()` из некогда успешной сервлетной логики кончается обескураживающим:

> `java.lang.IllegalStateException`: No thread-bound request found: Are you referring to request attributes outside of an actual web request, or processing a request outside of the originally receiving thread? If you are actually operating within a web request and still receive this message, your code is probably running outside of DispatcherServlet: In this case, use RequestContextListener or RequestContextFilter to expose the current request.

Несмотря на подробную ошибку, ни один из предлагаемых ею классов здесь не поможет, потому что они работают только на сервлетном стеке.

## Решение

В отличие от случая с MDC, здесь не удастся обойтись лишь добавлением какого-то нового класса/метода, который сделает “всё хорошо”. Придётся залезть в ту логику, которая обращается к `RequestContextHolder'`у, и адаптировать её к работе одновременно на двух стеках: сервлетном и реактивном. Крупными мазками эта адаптация состоит в следующем:

1. Избавиться от статической зависимости
2. Перейти на обобщённый класс запроса
   1. Унифицировать получение данных запроса

3. Обеспечить доступность “реактивного” запроса из единой точки обращения.

Пункт 2.1 является следствием пункта 2, поэтому сделан вложенным. Ниже приводятся конкретные шаги и объяснения, составляющие суть каждого пункта.

### Удаление статической зависимости

Даже безотносительно к решаемой здесь задаче переход от вызова статического метода к вызову instance-метода у Spring-бина – хорошая идея как с точки зрения тестируемости кода (см. минутку занудства выше), так и с точки зрения наведения порядка, ведь это позволит сделать зависимость от такого метода более явной и управляемой.

В случае с классом `RequestContextHolder` можно начать с простого и обернуть его в специально заведённый для этого бин, например, такой:

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

И хотя для модульных тестов это уже большое благо, для адаптации к реактивному стеку сделанная правка – условие необходимое, но не достаточное. Нужно ещё как-то преобразовать тип результата (`HttpServletRequest`), чтобы он не был так сильно привязан к сервлетному стеку годился для стека реактивного.

### Обобщение класса запроса

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

### Извлечение данных запроса

Найденная абстракция хороша тем, что под неё подпадают запросы из обоих миров, однако её одной мало, потому что состав методов интерфейса `HttpRequest` (даже с учётом наследования) весьма скуден:

{{< figure src="img/http-request-methods.png" caption="Методы интерфейса HttpRequest" numbered="false" >}}

В нём нет методов для получения многих часто используемых данных запроса, например:

* URI-параметров (аналог `javax.servlet.ServletRequest#getParameter`),
* cookies (аналог `javax.servlet.http.HttpServletRequest#getCookies`),
* атрибутов (аналог `javax.servlet.ServletRequest#getAttribute`).

Нет в нём и аналога метода `ServletRequest#getInputStream` для чтения тела запроса. Да и вообще работа с телами запросов/ответов в реактивном стеке существенно отличается от сервлетного и заслуживает отдельной заметки. А здесь для примера будут рассмотрено извлечение только приведенные выше данных.

Пользуясь выделенным ранее общим интерфейсом `HttpRequest` и осознанием того, что какие-то данные запроса придётся извлекать отдельными методами, можно уже сейчас наметить примерный скелет этих методов. Для простоты положим, что они будут находиться в том же классе `HttpRequestAccessor`, а выглядеть согласно такому шаблону:

```java
public Object getSomethingFrom(HttpRequest request) {
  // если запрос сервлетный
  if (request instanceof ServletServerHttpRequest wrapper) {          // [1]
    HttpServletRequest servletRequest = wrapper.getServletRequest();  // [2]
    // извлекаем что-то из сервлетного запроса
    return something;
  }
  // если запрос реактивный
  if (request instanceof ServerHttpRequest reactiveRequest) {         // [3]
    // извлекаем что-то из реактивного запроса
    return something;
  }
  // если запрос - неведомая дичь
  throw new IllegalArgumentException("Неизвестный тип запроса: " + request.getClass());
}
```

 :one: Используем [Pattern Matching](https://docs.oracle.com/en/java/javase/16/language/pattern-matching-instanceof-operator.html) (JDK 16+) для  краткости.  
:two: Избавляемся от обёртки.  
:three: `ServerHttpRequest` – это интерфейс `org.springframework.http.server.reactive.ServerHttpRequest` (не путать с однофамильцем).

#### Извлечение URI-параметров

Семейство методов `ServletRequest#getParameter*` в сервлетном стеке умеет работать как с обычными query-параметрами, передаваемыми в URL после знака `?`, так и с параметрами POST-запросов, передаваемыми в теле (например, при сабмите HTML-формы с `Content-Type: application/x-www-form-urlencoded`). В реактивном стеке для этого введён отдельный метод `ServerWebExchange#getFormData`. Очевидно, в таких случаях требуется парсинг тела запроса, который остаётся за скобками рассмотрения. А в этом пункте речь пойдёт только об обычных советских URI-параметрах, используемых чаще всего с HTTP-методом `GET`.

В случае с “**реактивным**” запросом всё предельно просто – есть метод `ServerHttpRequest#getQueryParams`, который возвращает `MultiValueMap<String, String>` – Spring’овый аналог коллекции `Map<String, List<String>>`. Такое нагромождение коллекций нужно для того, чтобы учесть случай, когда в запросе присутствует параметр, указанный более одного раза, например:

```http
GET http://localhost:8081/inspect?rid=123&rid=567
```

В этом случае значения `123` и `567` станут элементами списка, доступного по ключу `rid`. Кажется, что это вполне удобный формат для представления результата, поэтому он и будет принят за основной далее.

А у “**сервлетных**” запросов аналогичный метод `ServletRequest#getParameterMap` несколько отличается: он не только умеет возвращать параметры из тела POST-запросов, но и делает это в другом виде – `Map<String, String[]>`, поэтому придётся адаптировать этот вид к принятому выше `MultiValueMap<String, String>`.

С учётом этих договорённостей метод получения URI-параметров в классе `HttpRequestAccessor` может выглядеть примерно так:

```java
public MultiValueMap<String, String> getParameters(HttpRequest request) {
  // если запрос сервлетный
  if (request instanceof ServletServerHttpRequest wrapper) {
    HttpServletRequest servletRequest = wrapper.getServletRequest();
    Map<String, String[]> parameterMap = servletRequest.getParameterMap();
    Map<String, List<String>> rawMap = parameterMap.entrySet().stream()  // [1]
      .map(entry -> Map.entry(entry.getKey(), Arrays.asList(entry.getValue())))
      .collect(toMap(Map.Entry::getKey, Map.Entry::getValue));
    return CollectionUtils.toMultiValueMap(rawMap);                      // [2]
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
:three: Если бы так было везде, эта статья бы не появилась.

#### Извлечение cookies

С печеньками всё несколько проще, хотя отличия всё-таки есть. Во-первых, аналогичные по смыслу методы

```text
javax.servlet.http.HttpServletRequest#getCookies
org.springframework.http.server.reactive.ServerHttpRequest#getCookies
```

в сервлетном и реактивном стеках соответственно возвращают коллекции разных классов:

```text
javax.servlet.http.Cookie
org.springframework.http.HttpCookie
```

Во-вторых, эта разница сводится не только к именам, но и к предназначениям классов (и, как следствие, к составу их полей). Класс `javax.servlet.http.Cookie` подходит как для cookies в *запросах* (заголовок `Cookie`), так и в *ответах* (заголовок `Set-Cookie`), поэтому содержит много полей. А класс `org.springframework.http.HttpCookie` описывает cookies только в *запросах*, поэтому содержит лишь поля `name` и `value`. Для ответов в Spring Web выделен отдельный класс `ResponseCookie`, но поскольку здесь речь идёт только о запросах, он нам не пригодится.

Классы для cookies в обоих стеках следят за валидностью своих данных по [RFC 6265](https://tools.ietf.org/html/rfc6265).

Решающим критерием выбора между этими классами, очевидно, является пригодность к применению с любым стеком, и здесь безусловно выигрывает класс `HttpCookie`, потому что он декларирован в независимом от стека модуле Spring Web. Этот класс и возьмём. Тогда получение линейного списка cookies можно оформить примерно в такой метод:

```java
public List<HttpCookie> getCookies(HttpRequest request) {
  // если запрос сервлетный
  if (request instanceof ServletServerHttpRequest servletRequest) {
    Cookie[] cookies = servletRequest.getServletRequest().getCookies();
    if (cookies == null) {
      return List.of();
    }
    return Arrays.stream(cookies)
      .map(servletCookie ->              // [1]
           new HttpCookie(servletCookie.getName(), servletCookie.getValue()))
      .toList();
  }
  // если запрос реактивный
  if (request instanceof ServerHttpRequest reactiveRequest) {
    return reactiveRequest.getCookies()
      .values().stream()
      .flatMap(List::stream)             // [2]
      .toList();
  }
  // если запрос - неведомая дичь
  throw new IllegalArgumentException("Неизвестный тип запроса: " + request.getClass());
}
```

:one: Превращаем “сервлетные” cookies в абстрактные.   
:two: Для простоты дальнейшей демонстрации превращаем `MultiValueMap<String, HttpCookie>` в линейный список `List<HttpCookie>` (хотя в production коде это было бы, скорее, вредительством).

#### Извлечение атрибутов

В отличие от параметров/заголовков/cookies/тела, атрибуты не являются частью протокола HTTP. Это своего рода внутренний механизм серверных фреймворков для передачи мета-данных между этапами обработки запроса внутри сервера. Соответственно, и имплементация этого механизма может отличаться от фреймворка к фреймворку. Так, например, в Spring WebFlux нет понятия “атрибуты **запроса**”, вместо этого там используются атрибуты **обмена** – сущности, объединяющей запрос и ответ в рамках одного такта взаимодействия между клиентом и сервером. Обмен представлен интерфейсом `org.springframework.web.server.ServerWebExchange`, у которого есть методы `getAttribute*()`. А в Servlet API для той же цели служат методы `javax.servlet.ServletRequest#getAttribute*()`. В обоих случаях тип значения атрибутов объявлен как `Object`, чтобы через них можно было передавать любые объекты, а не только строки. Это вполне допустимо благодаря тому, что наружу (клиентам) атрибуты, как правило, не отдаются.

Поскольку атрибуты могут использоваться как фреймворками, так и прикладной логикой, во избежание коллизий имена атрибутов должны следовать тем же правилам, что и имена Java-пакетов, в частности (выдержка из javadoc на `ServletRequest`):

> Names beginning with `java.*` and `javax.*` are reserved for use by the Servlet specification. Names beginning with `sun.*, com.sun.*, oracle.*` and `com.oracle.*`) are reserved for use by Oracle Corporation.

Этого придерживается и Spring – он предваряет имена своих вспомогательных атрибутов (например, `bestMatchingHandler` и `producibleMediaTypes`) полным именем класса `HandlerMapping`. И хотя имена и смысл самих атрибутов в Spring WebMVC и WebFlux почти полностью совпадают, фактические имена атрибутов **различаются**, потому что в каждом фреймворке есть **свой** класс `HandlerMapping`:

```text
org.springframework.web.servlet.HandlerMapping
org.springframework.web.reactive.HandlerMapping
```

Из этого следует, что просто извлекать атрибуты из разных мест для WebMVC и WebFlux недостаточно, нужно ещё учитывать их имена. Однако здесь важно не увлечься и не начать писать логику преобразования имён одного фреймворка в имена другого. Ведь, если задуматься, атрибуты каждого фреймворка должны запрашиваться только в рамках работы на его родном стеке. Другими словами, попытка запросить атрибут, например, с *сервлетным* именем при работе на *реактивном* стеке – это явный признак произошедшей путаницы. И в таком случае нужна не “умная” логика преобразования, а тревога, чтобы как можно раньше “подсветить” место потенциального источника проблем. В случае с сервлетным стеком какой-нибудь первоклассный программист (учащийся в первом классе) написал бы эту подсветку примерно так:

```java
Assert.doesNotContain(attributeName, "org.springframework.web.reactive.HandlerMapping",
  "В сервлетном режиме не должен запрашиваться реактивный атрибут " + attributeName);
```

, где `Assert` – это `org.springframework.util.Assert`.

Придерживаясь взятого ранее курса на возврат *полной линейной* коллекции запрашиваемых данных запроса, класс `HttpRequestAccessor` мог бы предоставлять для атрибутов вот такой метод:

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
    ServerWebExchange currentExchange = CURRENT_EXCHANGE_HOLDER.get();  // [3]
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
:three: Что такое `CURRENT_EXCHANGE_HOLDER` – см. в следующем пункте.

На этом методе завершается рассмотрение подходов к извлечению данных запроса, а вместе с ним – и второй из четырёх шагов по адаптации `RequestContextHolder`'ной логики к работе на двух стеках. Теперь нужно добиться повсеместной доступности текущего запроса, как это было на чисто сервлетном стеке. Этому и будет посвящён следующий (последний) шаг.

### Обеспечение доступности запроса

Чтобы можно было заполучить текущий запрос практически из любой точки ~~мира~~ прикладной логики, не пробрасывая его от сáмого контроллера, можно применить тот же подход, который был описан в [заметке про MDC]({{< relref "/post/2021/reactivlet-2-mdc" >}}). Суть подхода примерно в следующем:

1. Обеспечить сохранение запроса в поточно-локальную переменную в сáмом начале обработки.
   Это будет поток-обработчик HTTP-запросов в Reactor’е; такие потоки максимально быстро освобождаются под другие нужды, поэтому для сохраненного здесь запроса ещё нужно:
2. Обеспечить пересохранение в *ту же* поточно-локальную переменную, но уже *другого* потока, занимающегося прикладной логикой.

Основное отличие от случая с MDC заключается в том, что поточно-локальная переменная берётся не из фреймворка/библиотеки, а заводится вручную. Для примера она представлена упомянутым выше полем `CURRENT_EXCHANGE_HOLDER` в классе `HttpRequestAccessor`:

```java
static final ThreadLocal<ServerWebExchange> CURRENT_EXCHANGE_HOLDER =
  new InheritableThreadLocal<>();
```

, и у неё есть особенности:

* область видимости `package-private` позволяет получить к ней прямой доступ из соседнего класса (см. ниже), но при этом не делать её доступной публично;
* типом хранимого объекта является именно `ServerWebExchange`, а не `HttpRequest`, потому что в Spring WebFlux атрибуты привязаны не к запросу, а к обмену (см. пункт “*Извлечение атрибутов*”);
* тип самого поля выбран как `InheritableThreadLocal`, чтобы дочерние потоки тоже могли заполучить себе текущий запрос.

Эти особенности проливают свет на класс, который будет управлять такой переменной. Он будет расположен в том же самом пакете (для простоты) и будет иметь доступ к текущему обмену. Следовательно, он должен являться веб-фильтром в терминах Spring WebFlux. Кроме того, он же является и регистратором хука в терминах Project Reactor. А называться он будет `Reactor2ServletBridge` (почему бы и нет?).

Поскольку подробный вывод подобного класса уже приводился в [предыдущей части]({{< relref "/post/2021/reactivlet-2-mdc" >}}), здесь будет сразу представлен полный код класса:

```java
@Component
@ConditionalOnWebApplication(type = REACTIVE)
class Reactor2ServletBridge implements WebFilter, Ordered {

  @Override
  public int getOrder() {
    return (Ordered.LOWEST_PRECEDENCE - 100);  // [1]
  }

  @Override
  public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
    return chain.filter(exchange)              // [2]
        .doOnSubscribe(sub -> {              
          HttpRequestAccessor.CURRENT_EXCHANGE_HOLDER.set(exchange);
        })
        .doFinally(whatever -> {
          HttpRequestAccessor.CURRENT_EXCHANGE_HOLDER.remove();
        });
  }

  @PostConstruct                               // [3]
  void setupReactorThreadsDecorator() {
    Schedulers.onScheduleHook("REQUEST_HOLDER", runnable -> {
      ServerWebExchange currentExchange =      // [4]
        HttpRequestAccessor.CURRENT_EXCHANGE_HOLDER.get();
      return () -> {
        if (currentExchange != null) {         // [5]
          HttpRequestAccessor.CURRENT_EXCHANGE_HOLDER.set(currentExchange);
        }
        try {
          runnable.run();                      // [6]
        } finally {
          HttpRequestAccessor.CURRENT_EXCHANGE_HOLDER.remove();
        }
      };
    });
  }

  @PreDestroy                                  // [7]
  void shutdownReactorThreadsDecorator() {
    Schedulers.resetOnScheduleHook("REQUEST_HOLDER");
  }
}
```

:one: Делаем фильтр одним из первых в цепочке, чтобы при вызове из других фильтров сервлетная логика уже могла полагаться на текущий запрос.  
:two: Вкрапляемся в начало и конец работы реактивного конвейера.  
:three: Навешиваем хук после создания единственного экземпляра класса.  
:four: Это действие произойдёт в исходном потоке, от которого будет ответвлен новый.  
:five: Это действие произойдёт уже в ответвленном потоке.  
:six: Теперь целевая задача может полагаться на поточно-локальный запрос, как и раньше.  
:seven: Дерегистрируем хук при завершении контекста (для порядка).

{{% callout info %}} Приведённый класс обеспечивает присутствие запроса (обмена) в потоках, ответвлённых только планировщиками Project Reactor (см. [Schedulers](https://projectreactor.io/docs/core/release/reference/#schedulers)). Когда и если потребуется обеспечить его присутствие в других потоках, например, порождённых из `ContainerFactory` в Spring AMQP, для них придётся придумывать другое решение (вероятно, со схожим смыслом). {{% /callout %}}

Теперь осталось лишь правильно считать значение созданной поточно-локальной переменной. Поскольку стоит задача сделать её максимально доступной для прикладной логики, наиболее подходящим местом для чтения выглядит введённый ранее метод `HttpRequestAccessor#fetchCurrentRequest`, который может быть похож на:

```java
public HttpRequest fetchCurrentRequest() {
  ServerWebExchange currentReactiveExchange = CURRENT_EXCHANGE_HOLDER.get();
  if (currentReactiveExchange != null) {                   // [1]
    return currentReactiveExchange.getRequest();
  }
  var servletRequestAttributes = (ServletRequestAttributes)
    RequestContextHolder.currentRequestAttributes();       // [2]
  HttpServletRequest servletRequest = servletRequestAttributes.getRequest();
  return new ServletServerHttpRequest(servletRequest);     // [3]
}
```

:one: Если “реактивного” запроса нет, пытаемся проверить “сервлетный”.  
:two: Напомним, что при отсутствии “сервлетного” запроса, этот метод выбросит то самое `IllegalStateException`, о котором шла речь в [начале](#Поддержка-RequestContextHolder) раздела.   
:three: Оборачиваем “сервлетный” запрос, чтобы обеспечить его совместимость с абстрактным `HttpRequest`.

Это последний из методов, которых не хватало для работы задуманного ранее `HttpRequestAccessor`'а.

### Пример применения

В unit-тестах при необходимости описанный выше метод можно будет легко замокировать, абстрагировавшись таким образом от деталей получения запроса. А в production-коде он теперь может использоваться почти наравне с некогда популярным `RequestContextHolder#currentRequestAttributes()` с той лишь разницей, что теперь нужно сначала внедрить бин `httpRequestAccessor` и получать данные запроса (кроме заголовков) через него. С точки зрения пользовательской логики его применение может выглядеть примерно так:

```java
HttpRequest currentRequest = httpRequestAccessor.fetchCurrentRequest();
MultiValueMap<String, String> parameters = httpRequestAccessor.getParameters(currentRequest);
List<HttpCookie> cookies = httpRequestAccessor.getCookies(currentRequest);
Map<String, String> attributes = httpRequestAccessor.getAttributes(currentRequest);
```

Примечательно и важно, что этот фрагмент кода теперь можно использовать как на сервлетном, так и на реактивном стеке, никаких `if`'ов, условных бинов или полиморфизма больше не требуется.

Так, например, в прилагаемом [демо-проекте](https://github.com/Toparvion/reactivlet-sample){{< icon name="github" pack="fab" >}}  в модуле `shared` есть [метод](https://github.com/Toparvion/reactivlet-sample/blob/94eebf15d73761d3db102788d480ee4728be3e2b/shared/src/main/java/pro/toparvion/sample/reactivlet/shared/UnifiedController.java#L32) `UnifiedController#inspect()`, как раз содержащий такой фрагмент кода и возвращающий его результаты в виде:

```java
return Map.of(
  "parameters", parameters,
  "headers", currentRequest.getHeaders(),
  "cookies", cookies,
  "attributes", attributes);
```

Если вызвать этот метод посредством вот такого запроса к реактивному демо-приложению:

```http
GET http://localhost:8081/inspect?rid=123&sid=abc&rid=567
Accept: application/json
Cookie: jid=ABC; cookie2=val2
```

, то он вернёт примерно такой результат:

```json
{
  "headers": {
    "Accept": [
      "application/json"
    ],
    "Host": [
      "localhost:8081"
    ],
    "Cookie": [
      "cookie2=val2; jid=ABC"
    ]
  },
  "cookies": [
    {
      "name": "jid",
      "value": "ABC"
    },
    {
      "name": "cookie2",
      "value": "val2"
    }
  ],
  "attributes": {
    "org.springframework.web.reactive.HandlerMapping.bestMatchingHandler": "pro.toparvion.sample.gateway.gatewaydemo.shared.UnifiedController#inspect()",
    "org.springframework.web.reactive.HandlerMapping.bestMatchingPattern": "/inspect",
    "org.springframework.web.reactive.HandlerMapping.pathWithinHandlerMapping": "/inspect",
    "org.springframework.web.reactive.HandlerMapping.uriTemplateVariables": "{}",
    "org.springframework.web.server.ServerWebExchange.LOG_ID": "430d181c-1",
    "org.springframework.web.reactive.HandlerMapping.matrixVariables": "{}"
  },
  "parameters": {
    "rid": [
      "123",
      "567"
    ],
    "sid": [
      "abc"
    ]
  }
}
```

Будучи выполненным на сервлетном демо-приложении (порт 8080), *этот же* самый метод `inspect()` отработает *точно так же*, разве что набор атрибутов будет специфичным для сервлетного стека. В этом и заключается сила описанного подхода – он позволяет одному коду работать на разных стеках.

Полный исходный код описанных здесь классов приведен в пакете `shared` прилагаемого [демо-проекта](https://github.com/Toparvion/reactivlet-sample){{< icon name="github" pack="fab" >}}.

## Попутное резюме

В этой части рассмотрен один из способов унифицировать доступ к текущему обрабатываемому запросу. Наверняка есть и другие. Жаждущий справедливости читатель наверняка расскажет о них в комментариях ниже, а потом, вместе с остальными, сможет перейти к [следующей части]({{< relref "/post/2021/reactivlet-4-aspect" >}}) и почитать про другой аспект обсуждаемой задачи.

&nbsp;

---

###### О картинке
<sup>
<a href="https://commons.wikimedia.org/wiki/File:RIAN_archive_341194_Kursk_Nuclear_Power_Plant.jpg">RIA Novosti archive, image #341194 / Sergey Pyatakov / CC-BY-SA 3.0</a>, <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0</a>, через Викисклад
</sup>
