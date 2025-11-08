---
title: "Java Flight Recorder"
subtitle: "Тренинг по мониторингу и профилированию приложений с помощью JFR"
summary: "Тренинг по мониторингу и профилированию приложений с помощью JFR"
authors:
  - toparvion
tags:
  - jvm
  - jfr
  - профилирование
  - мониторинг
  - тренинг
categories:
  - Performance
trainings:
  - performance
date: 2024-06-10
lastmod: 2025-11-08T07:32:05+07:00
featured: false
draft: false
pager: true
show_breadcrumb: true

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Титульный слайд"
  focal_point: ""
  preview_only: false

links:
  - icon: book
    icon_pack: fas
    name: Запросить проведение
    url: https://forms.yandex.ru/u/66b620c5f47e7313456e26e4/
#  - icon: github
#    icon_pack: fab
#    name: Тестовый проект
#    url: https://github.com/Toparvion/spring-petclinic-rest

gallery_item:
- album: examples/training/jfr
  image: sample-slide-1.png
  caption: Варианты включения записи JFR
- album: examples/training/jfr
  image: sample-slide-2.png
  caption: Выбор предустановленных настроек записи
- album: examples/training/jfr
  image: sample-slide-3.png
  caption: Вариант порядка действий для записи JFR
- album: examples/training/jfr
  image: sample-slide-4.png
  caption: Заголовок к подразделу о JFR.view

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
### Описание

Проблемы с производительностью всегда стреляют в самые неподходящие моменты, и, каким бы совершенным не был код, они всё равно случаются. Значит, их нужно уметь отслеживать и выявлять заранее.

Но как это сделать, если типичный мониторинг показывает только “среднюю температуру по больнице”, а проводить глубокое профилирование на production – опасная затея? Было бы хорошо иметь внутри приложения некий “бортовой самописец”, по данным которого можно будет выявить приближающуюся аварию или хотя бы постфактум выяснить, что ей предшествовало, если она всё же случилась. К счастью, такой инструмент уже есть внутри Java-машины – **Java Flight Recorder** (JFR).

В этом тренинге вы узнаете, какие возможности предоставляет JFR, как использовать его в разных окружениях (от локальной машины до production) и как анализировать собираемые им данные для выявления узких и горячих мест в вашем коде. Вы также научитесь применять его в качестве альтернативы сторонним (часто отнюдь не бесплатным) профилировщикам и интегрировать его в свой код, чтобы получить “log.debug на стероидах”.

{{% callout note %}}
Формат тренинга описан на странице серии тренингов про [Производительность](/training/performance/).
{{% /callout %}}

### Содержание

#### Теория 🎓

1. Что такое Java Flight Recorder
   1. В чём идея
   2. Из чего состоит (части JVM и клиентская)
   3. Терминология
   4. Сколько это “стоит” по производительности
2. Управление записью JFR
   1. Общий порядок действий
   2. Способы запуска записи (4 шт.)
3. Знакомство с Mission Control
   1. Общие сведения
   2. Варианты дистрибутивов
   3. Конфигурация записи
   4. Отслеживание статуса записи
   5. Получение данных от JFR
4. Как читать записанные JFR-сессии:
   1. Основы интерфейса Mission Control
   2. Работа с временными развёртками событий
   3. Как и когда искать события в общем журнале
   4. Типичные задачи, решаемые с помощью JFR
5. Советы по применению JFR на production

#### Практика ⚒️

В этом тренинге практика состоит из двух частей:
1. Разбор трёх практический кейсов (аномальных поведений) в [лабораторном приложении](https://github.com/Toparvion/spring-petclinic-rest) Spring PetClinic REST при помощи JFR по следующей схеме:
   1. Описание бизнес-кейса
   2. Пояснения к реализаци
   3. Описание проблемы
   4. Анализ проблемы
   5. Устранение

2. Учебная доработка того же [лабораторного приложения](https://github.com/Toparvion/spring-petclinic-rest) с целью наделения его новыми функциями на основе кастомных событий JFR и программной подписки на события через JFR Streaming API.

### Примеры слайдов
Из теоретической части занятия
{{< gallery album="training/jfr" >}}

### Материалы в шпаргалке

* Показания к применению
* Управление записью
* Просмотр записей в консоли
* Встроенные режимы записи
* Устройство JFR

---

### Интересно?
Если хотите, чтобы я провёл этот тренинг в вашей компании, вы&nbsp;можете:
{{< cta cta_text="Оставить заявку" cta_link="https://forms.yandex.ru/u/66b620c5f47e7313456e26e4/" cta_new_tab="true" cta_alt_text="Задать вопрос" cta_alt_link="https://t.me/Toparvion" cta_alt_new_tab="true" >}}