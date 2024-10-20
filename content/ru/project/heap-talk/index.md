---
title: 💬 HeapTalk
summary: Прототип утилиты для анализа дампов памяти JVM с помощью ИИ
tags:
- отладка
- производительность
date: "2024-10-12T00:00:00Z"
weight: 10

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: ""
  focal_point: Smart

links:
  - icon: github
    icon_pack: fab
    name: Проект на GitHub
    url: https://github.com/Toparvion/heap-talk
---

**HeapTalk** – результат проверки гипотезы о том, можно ли искать данные в дампах памяти JVM, формулируя запросы на человеческом языке. Похоже, что **можно**.

### Предыстория

Идея создания этого инструмента родилась на пересечении двух фактов:

* для гибкого анализа дампов давно существуют движки исполнения SQL-подобных запросов, но их синтаксис весьма специфичен и потому сложен для применения;
* большие языковые модели ([LLM](https://ru.wikipedia.org/wiki/%D0%91%D0%BE%D0%BB%D1%8C%D1%88%D0%B0%D1%8F_%D1%8F%D0%B7%D1%8B%D0%BA%D0%BE%D0%B2%D0%B0%D1%8F_%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D1%8C)) хорошо зарекомендовали себя в качестве генераторов SQL-запросов, по сути став переводчиками с человеческого языка на SQL.

Основная сложность применения LLM к дампам памяти кроется в том, что нужно как-то “объяснить” языковой модели схему базы данных, по которой задаётся вопрос, то есть перечислить ей все таблицы, колонки, их типы и связи, чтобы она могла правильно сгенерировать SQL-запрос. Но если для традиционных реляционных СУБД это легко решается выгрузкой [DDL](https://ru.wikipedia.org/wiki/Data_Definition_Language)-выражений, описывающих схему, то для дампов памяти JVM такого способа нет, ибо сами дампы по своей природе реляционными базами, конечно же, не являются.

HeapTalk пытается решить эту проблему. Он парсит дамп памяти при помощи библиотеки [HeapLib](https://github.com/aragozin/heaplib), находит в нём все прикладные классы приложения (для этого пользователь указывает корневой пакет/пакеты своего приложения), а затем, анализируя их, составляет набор синтаксически корректных DDL-выражений, описывающих схему этой псевдо-БД для применения с  [MAT Calcite Plugin](https://github.com/vlsi/mat-calcite-plugin).

### Особенности

Помимо специфичных для JVM особенностей, основным отличием генерируемого DDL-скрипта от обычного является отсутствие явных описаний связей между таблицами, то есть никаких `add foreign key` там нет. Вместо этого для ссылочных колонок добавляется обычный текстовый комментарий, поясняющий соответствующую связь. Этого достаточно при работе, как минимум, с ChatGPT, потому что эта модель воспринимает DDL-скрипт скорее как человек, а не как SQL-движок.

После генерации DDL-скрипта HeapTalk вставляет его в специально заготовленный шаблон промпта для ChatGPT вместе с задаваемым вопросом и при помощи библиотеки [langchain4j](https://github.com/langchain4j/langchain4j) отправляет в [API OpenAI](https://openai.com/index/openai-api/). Ответ по умолчанию возвращается в чистом виде, чтобы утилиту можно было включать в состав какого-либо автоматизированного конвейера.

Для общения с API OpenAI пользователь должен предоставить соответствующий ключ **API Key**. Однако, поскольку в нынешних реалиях в РФ его не очень просто получить, в утилите предусмотрены две альтернативы:

* При указании опции `--api-key=demo` будет использован демо-ключ от библиотеки langchain4j. Его должно хватить для выполнения единичных запросов, но при повышении частоты обращений с ним очень легко упереться в ограничения;
* При указании флага `--proxy-api` запросы к ChatGPT будут проксироваться через российский сервис [ProxyApi](https://proxyapi.ru/) (соответственно, ключ в опции `--api-key` должен быть указан от этого сервиса, а не от OpenAI).

### Примеры запросов

Для дампа памяти, снятого с [модифицированной](https://github.com/Toparvion/spring-petclinic-rest) версии приложения [Spring PetClinic REST](https://github.com/spring-petclinic/spring-petclinic-rest), при помощи HeapTalk были сгенерированы запросы, приведённые ниже. Соответствующие ответы получены путем вставки сгенерированных запросов в плагин **Calcite SQL** в инструменте [Eclipse MAT](https://projects.eclipse.org/projects/tools.mat) и последующим экспортом результата в текстовый файл.

{{< spoiler text="DDL-скрипт дампа" >}}

```sql
-- table with entities of type 'Owner'
CREATE TABLE "org.springframework.samples.petclinic.model.Owner" (
    this INTEGER PRIMARY_KEY,    -- unique ID for each entity
    address VARCHAR(1024),
    city VARCHAR(1024),
    telephone VARCHAR(1024),
    pets INTEGER,    -- can be joined with "org.hibernate.collection.spi.PersistentSet" on 'this'
    firstName VARCHAR(1024),
    lastName VARCHAR(1024),
    id INTEGER
);

-- table with entities of type 'User'
CREATE TABLE "org.springframework.samples.petclinic.model.User" (
    this INTEGER PRIMARY_KEY    -- unique ID for each entity
);

-- table with entities of type 'Role'
CREATE TABLE "org.springframework.samples.petclinic.model.Role" (
    this INTEGER PRIMARY_KEY    -- unique ID for each entity
);

-- table with entities of type 'Pet'
CREATE TABLE "org.springframework.samples.petclinic.model.Pet" (
   this INTEGER PRIMARY_KEY,    -- unique ID for each entity
   birthDate INTEGER,    -- can be joined with "java.time.LocalDate" on 'this'
   type INTEGER,    -- can be joined with "org.springframework.samples.petclinic.model.PetType" on 'this'
   owner INTEGER,    -- can be joined with "org.springframework.samples.petclinic.model.Owner" on 'this'
   visits INTEGER,    -- can be joined with "org.hibernate.collection.spi.PersistentSet" on 'this'
   name VARCHAR(1024),
   id INTEGER
);

-- table with entities of type 'Vet'
CREATE TABLE "org.springframework.samples.petclinic.model.Vet" (
   this INTEGER PRIMARY_KEY         -- unique ID for each entity
);

-- table with entities of type 'Visit'
CREATE TABLE "org.springframework.samples.petclinic.model.Visit" (
    this INTEGER PRIMARY_KEY,    -- unique ID for each entity
    date INTEGER,    -- can be joined with "java.time.LocalDate" on 'this'
    description VARCHAR(1024),
    pet INTEGER,    -- can be joined with "org.springframework.samples.petclinic.model.Pet" on 'this'
    id INTEGER
);

-- table with entities of type 'Person'
CREATE TABLE "org.springframework.samples.petclinic.model.Person" (
    this INTEGER PRIMARY_KEY    -- unique ID for each entity
);

-- table with entities of type 'BaseEntity'
CREATE TABLE "org.springframework.samples.petclinic.model.BaseEntity" (
    this INTEGER PRIMARY_KEY    -- unique ID for each entity
);

-- table with entities of type 'Specialty'
CREATE TABLE "org.springframework.samples.petclinic.model.Specialty" (
    this INTEGER PRIMARY_KEY    -- unique ID for each entity
);

-- table with entities of type 'PetType'
CREATE TABLE "org.springframework.samples.petclinic.model.PetType" (
    this INTEGER PRIMARY_KEY,    -- unique ID for each entity
    name VARCHAR(1024),
    id INTEGER
);

-- table with entities of type 'NamedEntity'
CREATE TABLE "org.springframework.samples.petclinic.model.NamedEntity" (
    this INTEGER PRIMARY_KEY    -- unique ID for each entity
);
```

{{< /spoiler >}}

<br/>

#### Пример 1

*Вопрос пользователя:*

> Which pets (listed by names) were born after 2011?

*Сгенерированный запрос*:

```sql
SELECT p.name
FROM "org.springframework.samples.petclinic.model.Pet" p
JOIN "java.time.LocalDate" ld ON p.birthDate = ld.this
WHERE ld."year" > 2011
```

*Результат выполнения запроса:*

```markdown
name
----------------
Samantha
Max
Basil
Sly
```

<br/>

#### Пример 2

*Вопрос пользователя:*

> What's the phone number of the person whose hamster named Basil?

*Сгенерированный запрос*:

```sql
SELECT o.telephone
FROM "org.springframework.samples.petclinic.model.Owner" o
JOIN "org.springframework.samples.petclinic.model.Pet" p ON o.this = p.owner
WHERE toString(p.name) = 'Basil'
```

*Результат выполнения запроса:*

```markdown
telephone
--------------
6085551749
```

<br/>

#### Пример 3

*Вопрос пользователя:*

> How many snakes are registered in our database?

*Сгенерированный запрос*:

```sql
SELECT COUNT(*)
FROM "org.springframework.samples.petclinic.model.Pet" p
JOIN "org.springframework.samples.petclinic.model.PetType" pt ON p.type = pt.this
WHERE toString(pt.name) = 'snake'
```

*Результат выполнения запроса:*

```markdown
             EXPR$0
--------------------
                  1
```

<br/>

#### Пример 4

*Вопрос пользователя:*

> Who the Max cat belongs to?

*Сгенерированный запрос*:

```sql
SELECT o.firstName, o.lastName
FROM "org.springframework.samples.petclinic.model.Owner" o
JOIN "org.springframework.samples.petclinic.model.Pet" p ON o.this = p.owner
JOIN "org.springframework.samples.petclinic.model.PetType" pt ON p.type = pt.this
WHERE toString(p.name) = 'Max' AND toString(pt.name) = 'cat'
```

*Результат выполнения запроса:*

```markdown
firstName     |lastName
-----------------------
Jean          |Coleman
```

<br/>

#### Пример 5

*Вопрос пользователя:*

> Who is the oldest animal in our clinic?

*Сгенерированный запрос*:

```sql
SELECT p.name, toString(pt.name) AS type, MIN(bd."year") AS birthYear
FROM "org.springframework.samples.petclinic.model.Pet" p
JOIN "java.time.LocalDate" bd ON p.birthDate = bd.this
JOIN "org.springframework.samples.petclinic.model.PetType" pt ON p.type = pt.this
GROUP BY p.name, toString(pt.name)
ORDER BY MIN(bd."year") ASC
FETCH FIRST 1 ROWS ONLY
```

*Результат выполнения запроса:*

```markdown
name          |type| birthYear
-------------------------------
Mulligan      |dog |      2007
```

<br/>

#### Пример 6

*Вопрос пользователя:*

> What is the distribution of animal species in our clinic?

*Сгенерированный запрос*:

```sql
SELECT toString(pt.name) AS species, COUNT(p.this) AS "count"
FROM "org.springframework.samples.petclinic.model.Pet" p
JOIN "org.springframework.samples.petclinic.model.PetType" pt ON p.type = pt.this
GROUP BY toString(pt.name)
ORDER BY "count" DESC
```

*Результат выполнения запроса:*

```markdown
species         | count
------------------------
cat             |     4
dog             |     4
bird            |     2
hamster         |     1
snake           |     1
lizard          |     1
```

:warning: Кавычки вокруг слова “count” пришлось добавить вручную.

<br/>

#### Пример 7

*Вопрос пользователя:*

> In which city are the most animals registered?

*Сгенерированный запрос*:

```sql
SELECT toString(o.city) AS city, COUNT(p.this) AS animal_count
FROM "org.springframework.samples.petclinic.model.Owner" o
JOIN "org.springframework.samples.petclinic.model.Pet" p ON o.this = p.owner
GROUP BY toString(o.city)
ORDER BY animal_count DESC
LIMIT 1
```

*Результат выполнения запроса:*

```markdown
city          | animal_count
-----------------------------
Madison       |            4
```

<br/>

#### А дальше

Если вам понравился этот проект или появились вопросы, вы можете:

{{< cta cta_text="Поставить ⭐ на GitHub" cta_link="https://github.com/Toparvion/heap-talk" cta_new_tab="true" cta_alt_text="Задать вопрос" cta_alt_link="https://t.me/Toparvion" cta_alt_new_tab="true" >}}
