---
title: "Профилирование"
subtitle: "Тренинг профилированию приложений на JVM"
summary: "Тренинг профилированию приложений на JVM"
authors:
  - toparvion
tags:
  - jvm
  - profiling
  - sampling
  - tracing
categories:
  - Performance
trainings:
  - performance
date: 2024-06-01
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
- album: examples/training/profiling
  image: sample-slide-1.png
  caption: Различие измеряемых времён в профилировании
- album: examples/training/profiling
  image: sample-slide-2.png
  caption: Краткий экскурс в устройство аллокации памяти в JVM
- album: examples/training/profiling
  image: sample-slide-3.png
  caption: Как подлючать async-profiler через командную строку
- album: examples/training/profiling
  image: sample-slide-4.png
  caption: Пример flame graph с неожиданной нативной частью

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

### Описание

Когда ваше приложение тормозит, да ещё на production, важно уметь быстро ответить на два вопроса: где узкое место и почему оно тормозит? Для этого, как правило, применяют профилирование. Вот только инструментов для него, мягко говоря, не один, и отличаются они чуть ли не до противоположности: платные и бесплатные, “из коробки” и подключаемые, CLI и GUI. При этом все они используются, а значит, каждый в чём-то хорош.

Во всем этом вам поможет разобраться предлагаемый тренинг. На нём вы узнаете о различных подходах к профилированию и их особенностях, научитесь оценивать применимость различных профайлеров, а главное – использовать их по назначению в зависимости от решаемой задачи.

{{% callout note %}}
Формат и стоимость тренинга приведены на странице серии тренингов про [Производительность](/training/performance/).
{{% /callout %}}

### Содержание

#### Теория 🎓

1. Введение в тему
   1. Что, когда и зачем профилировать?
   2. Способы профилирования “на пальцах”
      1. Sampling
      2. Tracing / instrumentation
      3. Events-based
   3. Сравнение подходов
   4. Варианты представления результатов для анализа
2. Поход по популярным инструментам
   1. Async-profiler
   2. VisualVM
   3. YourKit
   4. JFR
3. Советы про профилированию в различных окружениях

#### Практика ⚒️

Разбор проблем производительности на примере [лабораторного приложения](https://github.com/Toparvion/spring-petclinic-rest) Spring PetClinic REST по схеме:

1. Описание кейса
2. Имплементация (пояснения к реализации)
3. Проблема
4. Обнаружение (подсказки к проведению анализа)
5. Причина

### Примеры слайдов
Из теоретической части занятия
{{< gallery album="training/profiling" >}}

---

### Интересно?
Если хотите, чтобы я провёл этот тренинг в вашей компании, вы&nbsp;можете:
{{< cta cta_text="Оставить заявку" cta_link="https://forms.yandex.ru/u/66b620c5f47e7313456e26e4/" cta_new_tab="true" cta_alt_text="Задать вопрос" cta_alt_link="https://t.me/Toparvion" cta_alt_new_tab="true" >}}