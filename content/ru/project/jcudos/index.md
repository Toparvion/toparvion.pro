---
title: jCuDoS
summary: Инструментарий для применения AppCDS к микросервисам на Spring Boot *[в работе]*
tags:
- производительность
- appcds
- spring-boot
date: "2019-11-08T00:00:00Z"
weight: 30

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: ""
  focal_point: Smart

links:
  - icon: github
    icon_pack: fab
    name: Проект на GitHub
    url: https://github.com/Toparvion/jcudos
---

**jCuDoS** – это набор CLI-команд для упрощения и автоматизации применения AppCDS ко множеству различных приложений на Spring Boot (как правило, к парку микросервисов одного проекта). Команды могут вызываться как по отдельности, так и все вместе в определенном порядке. В последнем случае они реализуют приведенный ниже алгоритм.

## Алгоритм подготовки AppCDS для парка микросервисов

### Предусловия 

* Имеется множество “толстых” JAR от микросервисов на Spring Boot
* Все JAR-файлы имеют общий корень файловой системы
* Все микросервисы были хотя бы раз запущены с опцией `-XX:DumpLoadedClassList`

### Входные данные

* Glob к спискам классов, полученным опцией `-XX:DumpLoadedClassList`
* Glob к “толстым” JAR
* Путь к общей выходной директории (желаемый, например, `_appcds`)

### Процесс

#### A. Обработка списков классов

1. Выявить общую часть
1. Сохранить ее в `_appcds/_shared/list/classes.list` :zero:

#### B. Обработка каждой “толстой” JAR

1. Определить имя стартового класса.

   Если определить не удалось, пропустить обработку архива.

   Запомнить имя приложения как “голое” имя класса в нижнем регистре (далее – `<appName>`)

1. Сохранить имя стартового класса в `_appcds/<appName>/start-class.txt` :one:

1. Распаковать либы каждой толстой JAR в `_appcds/<appName>/lib/`

1. Сконвертировать толстую JAR в `_appcds/<appName>/lib/<appName>.slim.jar` :two:

#### C. Создание общего архива

1. Выявить общую часть либ по листингам директорий в `_appcds/<appName>/lib/`

1. Скопировать все выявленные либы в `_appcds/_shared/lib/`

1. Составить файл `_appcds/_shared/list/classpath.arg` :five: из абсолютных путей к скопированным либам

1. Запомнить список абсолютных путей к общим либам :three:

1. Перейти в директорию `_appcds` и выполнить там:

   ```bash
   java -Xshare:dump \
        -XX:SharedClassListFile=_shared/list/classes.list \
        -XX:SharedArchiveFile=_shared/jsa/classes.jsa \
        @_shared/list/classpath.arg &> log/dump.log
   ```

   Здесь пригодятся:  `_appcds/_shared/list/classes.list` :zero: и `_appcds/_shared/list/classpath.arg` :five:

#### D. Подготовка к запуску микросервисов

1. Удалить из директорий `_appcds/<appName>/lib/` либы по именам, взятым из списка общих либ :three:

1. Составить файл `_appcds/<appName>/appcds.arg` :four: из:

   1. опции `-XX:SharedArchiveFile=_appcds/_shared/jsa/classes.jsa`

   1. опции `-classpath`, состоящей из:

      1. списка общих либ :three:

      1. списка своих либ, взятого из листинга директории `_appcds/<appName>/lib/`

         Туда должна будет входить и тонкая JAR `_appcds/<appName>/lib/<appName>.slim.jar` :two:

      1. стартового класса :one:

#### Запуск микросервиса с AppCDS (за рамками автоматизации)

* Заменить в скрипте запуска опцию `-jar <appName>.jar` на `@appcds.arg` :four:

## Статус проекта

Основная разработка jCuDoS завершена, я опробовал его на тестовом парке микросервисов и получил предварительные результаты. Для полноценного применения его необходимо более тщательно протестировать и снабдить документацией. 

{{% callout info %}}
Если вы заинтересованы в применении этого инструмента или хотите поучаствовать в его развитии, [сообщите](/#contact) мне об этом, пожалуйста, и я постараюсь помочь.

{{% /callout %}}