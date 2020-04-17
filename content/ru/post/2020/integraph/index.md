---
title: "Как визуализировать граф Spring Integration в Neo4j?"
subtitle: "Про мостик из EIP в графовые СУБД"
summary: ""
authors: 
  - toparvion
tags:
  - spring-integration
  - neo4j
  - spring-boot
categories:
  - graph-storages
date: 2020-04-14T08:31:05+07:00
lastmod: 2020-04-14T08:31:05+07:00
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

Фреймворк Spring Integration [умеет](https://docs.spring.io/spring-integration/docs/5.2.5.RELEASE/reference/html/system-management.html#integration-graph) отдавать текущее состояние всех [EIP](https://www.enterpriseintegrationpatterns.com/)-компонентов и их связей в виде JSON-графа. Это кажется очень удобным для изучения и отладки, но увы, ни один из нагугливаемых инструментов (коих всего-то [раз](https://github.com/spring-projects/spring-flo/tree/angular-1.x/samples/spring-flo-si) /[два](https://ordina-jworks.github.io/architecture/2018/01/27/Visualizing-your-Spring-Integration-components-and-flows.html)) не даёт достаточной гибкости для визуализации и анализа такого графа. В этой заметке я покажу, как решить эту проблему путем импорта графа в графовую СУБД [Neo4j](https://neo4j.com/), где такая гибкость стоит на первом месте.

### Короче (tl;dr;)

Если некогда/не охота вникать, то вот что надо сделать:

1. **Установить** библиотеку APOC в свой экземпляр Neo4j.
1. Выполнить **этот** Cypher-скрипт, поменяв в нём URL на свой.

Ну, а если просто так не завелось или стали интересны подробности, то добро пожаловать дальше.

### Как получить описание графа в JSON?

Есть 2 не взаимоисключающих способа:

1. Если используется модуль `spring-integration-http` (или `spring-integration-webflux`), то граф можно получить, дёрнув [Integration Graph Controller](https://docs.spring.io/spring-integration/docs/5.2.5.RELEASE/reference/html/system-management.html#integration-graph-controller).
1. Если используется модуль `spring-integration-core` **вместе** со Spring Boot Actuator, то граф можно получить через его endpoint под названием [integrationgraph](https://docs.spring.io/spring-boot/docs/2.2.5.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints) (который [по умолчанию](https://docs.spring.io/spring-boot/docs/2.2.5.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints) не доступен через web).

В обоих случаях ответ будет выглядеть примерно так:

```json
{
  "contentDescriptor": {
    "providerVersion": "5.2.4.RELEASE",
    "providerFormatVersion": 1.1,
    "provider": "spring-integration",
    "name": "analog"
  },
  "nodes": [
    {
      "nodeId": 3,
      "componentType": "ExecutorSubscribableChannel",
      "name": "brokerChannel"
    },
    // ...
  ],
  "links": [
    {
      "from": 19,
      "to": 8,
      "type": "output"
    },
    // ...
  ]
}
```

На верхнем уровне ответа всегда только 3 поля:

- Объект `contentDescriptor` – общий описатель графа; поле `name` в нём берется из имени приложения (свойство `spring.application.name`). 
- Массив `nodes` содержит описания узлов графа – собственно EIP-компонентов, бины которых и составляют интеграционный конвейер. Наряду с именем `name`, типом `componentType` и идентификатором `nodeId`, в него также могут входить метрики и другие данные, но нас они не интересуют.
- Массив `links` описывает рёбра графа, т.е. связи между EIP-компонентами. Поля `from` и `to` указывают на значения `nodeId` исходящего и входящего узлов соответственно. А поле `type` содержит тип связи, коих всего 5: `input`, `output`, `route`, `error`, `discard`.

:point_up: *Каналы в терминах EIP **не** являются связями в графе!*  
Вместо этого они представлены такими же узлами, как например, адаптеры и фильтры, разве что их `componentType` обычно заканчивается словом *channel*.

Подробнее модель графа описана в [этом разделе](https://docs.spring.io/spring-integration/docs/5.2.5.RELEASE/reference/html/system-management.html#integration-graph) документации на Spring Integration.

#### На всякий случай

Прежде чем дёргать граф прямо из Neo4j, настоятельно рекомендую получить его чем-то попроще, например, обычным браузером, перейдя по ссылке [http://localhost:8080/actuator/integrationgraph](http://localhost:8080/actuator/integrationgraph) (для случая со Spring Boot Actuator). Если ответ не похож на приведенный выше JSON, то нет смысла двигаться дальше, нужно разобраться здесь.

Чаще всего проблема либо в ограничениях [CORS](https://ru.wikipedia.org/wiki/Cross-origin_resource_sharing), либо в недоступности отдающего граф компонента. Если приложение на Spring Boot (+Actuator) и развернуто локально, то обе проблемы можно решить добавлением в его настройки следующих [строк](https://docs.spring.io/spring-boot/docs/2.2.5.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-cors):

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
      cors:
        allowed-origins: '*'
        allowed-methods: '*'
```

*(только не надо делать так на production :pray: )*

А если приложение на чистом Spring Integration, то см. подсказки на [этой странице](https://docs.spring.io/spring-integration/docs/5.2.5.RELEASE/reference/html/system-management.html#integration-graph-controller) документации.

### Как импортировать граф в Neo4j

#### Входные требования

Чтобы залить граф в Neo4j, нам понадобится 2 вещи:

1. Собственно **СУБД Neo4j** *(неожиданно, правда?)*  
   Подойдёт любая бесплатная поставка:

   - Настольная [Neo4j Desktop](https://neo4j.com/download-center/#desktop)
   - Серверная [Neo4j Community Server](https://neo4j.com/download-center/#community)
   - Облачная песочница [Neo4j Sandbox](https://neo4j.com/sandbox/)  
     *Не требует инсталляции*

1. Библиотека полезных Cypher-процедур **APOC**

   Это де-факто стандартная библиотека от разработчиков Neo4j, поэтому её можно найти и поставить прямо с их [официального сайта](https://neo4j.com/docs/labs/apoc/current/introduction/#installation). В облачной песочнице Neo4j Sandbox она уже предустановлена.

При написании статьи использовалась Neo4j версии **4.0.1**, но поскольку на внутренности СУБД мы здесь не полагаемся, всё должно работать и на других версиях.[^1] При установке библиотеки APOC важно, чтобы первые 2 цифры её версии совпадали с такими же цифрами у самой Neo4j.

#### Подход 1: прямая визуализация

Поскольку граф Spring Integration, судя по его JSON-модели, содержит всё необходимое для визуализации, можно просто взять его “как есть” (узлы – к узлам, связи – к связям, свойства – к свойствам) и залить в графовую СУБД. Из всех свойств узлов мы возьмём только основные: `nodeId`, `nodeName` и `componentType`. Они поставляются базовым классом `org.springframework.integration.graph.IntegrationNode`, поэтому (наверно) должны присутствовать у всех без исключения узлов.

Основная идея в том, чтобы прямо из Neo4j сказать что-то вроде: 

> *Возьми JSON вот по **этому** URL, обойди вот **такие** его поля и разложи их данные по вот **таким** вершинам и рёбрам вот с **такими** свойствами.*

На языке Cypher это распоряжение может выглядеть примерно так:[^2]

```cypher
WITH "http://localhost:8083/actuator/integrationgraph" AS url
CALL apoc.load.json(url) YIELD value	// (1)
WITH value AS json, value.contentDescriptor AS jsonDescriptor
// descriptor (2)    
MERGE (descriptor:Descriptor {name: jsonDescriptor.name}) 
    ON CREATE SET 
    descriptor.providerVersion = jsonDescriptor.providerVersion, 
    descriptor.providerFormatVersion = jsonDescriptor.providerFormatVersion,
    descriptor.provider = jsonDescriptor.provider
// nodes (3)
WITH json, descriptor
UNWIND json.nodes AS jsonNode
MERGE (node:Node {nodeId: jsonNode.nodeId})
    ON CREATE SET
    node.componentType = jsonNode.componentType,
    node.name = jsonNode.name
// links (4)
WITH json, descriptor, node 
UNWIND json.links AS jsonLink
MATCH (a:Node {nodeId: jsonLink.from}), (b:Node {nodeId: jsonLink.to})
MERGE (a)-[:Link {type: jsonLink.type}]->(b)
// result (5)
RETURN descriptor, node
```







---

- [ ] Особенности развертывания в облачной песочнице упомянуты:
  - ~~Версия 3.5.11~~
  - Недоступность загрузки с `localhost`. Альтернатива:
    https://deploy-preview-1--toparvion.netlify.app/export/analog.json
  - Урезанная сигнатура методов `merge`
- [ ] Стиль оформления выложен и учтён в тексте

[^1]: В облачной песочнице на дату 16.04.20 использовалась Neo4j версии 3.5.11, для которой некоторые процедуры APOC имели отличия в сигнатурах от версии 4.0. Подробнее об этом см. ниже. 
[^2]: Если вставить этот запрос в облачную песочницу Neo4j Sandbox, он не выполнится, потому что для удалённого сервера Neo4j адрес `localhost` указывает на него самого, а не на эту машину. Чтобы всё-таки проверить запрос в песочнице, придётся либо развернуть своё Spring Integration приложение с доступным извне адресом, либо воспользоваться моим [примером](export/analog.json) готового JSON-графа от приложения [АнаЛог](/project/analog).  