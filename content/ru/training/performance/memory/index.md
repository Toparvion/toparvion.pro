---
title: "Анализ памяти"
subtitle: "Тренинг по работе с дампами памяти"
summary: "Тренинг по работе с дампами памяти"
authors:
  - toparvion
tags:
  - jvm
  - memory
  - dump
  - mat
  - oom
categories:
  - Performance
trainings:
  - performance
date: 2024-06-20
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
- album: examples/training/memory
  image: sample-slide-1.png
  caption: Сопоставление памяти в куче и вне её
- album: examples/training/memory
  image: sample-slide-2.png
  caption: Как применять jcmd для снятия дампа памяти
- album: examples/training/memory
  image: sample-slide-3.png
  caption: Цитата из блога А. Шипилёва о размерах объкетов в куче
- album: examples/training/memory
  image: sample-slide-4.png
  caption: Различия размеров объектов

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

### Описание

_“Это Java памяти много ест. Надо ей ещё добавить”_ – типичная ~~отмазка~~ резолюция от разработчиков, которым некогда/неохота/тяжело разбираться с истинной причиной очередного `OutOfMemoryError` на production. Повезло, если такое предложение прокатит и не повторится, но, скоре всего, рано или поздно разбираться всё же придётся.

В таких случаях, как правило, нужно сделать **снимок памяти** (heap dump), но с ним не всё так просто: он может много весить, его бывает не легко достать с production, в него не посмотришь “глазами” (он бинарный), а в спец. инструментах данных столько, что можно утонуть.

На этом треннинге вы научитесь снимать дампы памяти разными способами (в зависимости от окружения), ускорять и упрощать их передачу к себе для анализа и, главное, овладеете техниками и инструментами для эффективного обнаружения источников проблем с потреблением памяти: от построения наглядных отчётов до глубокого анализа графа объектов через язык запросов.

{{% callout note %}}
Формат и стоимость тренинга приведены на странице серии тренингов про [Производительность](/training/performance/).
{{% /callout %}}

### Содержание

#### Теория 🎓

1. Что такое дамп памяти JVM
   1. И чем он не является
   2. Связь с core dump
2. Предостережения о дампах памяти
   * (Безопасность и обфускация дампов)
3. Способы получения
   1. Внешними инструментами (4 шт.)
   2. Инструментами JDK (4 шт.)
   3. Алгоритм выбора инструмента
   4. Как сделать снятие дампов автоматическим
4. Основы устройства дампов:
   1. Особенности размеров объектов в памяти JVM
   2. GC roots
   3. Shallow size VS retained size
   4. Дерево доминаторов
5. Анализ дампов в Eclipse MAT
   1. Основные данные из дампа в Eclipse MAT
   2. Базовые приёмы навигации по куче
   3. Продвинутые приёмы (включая OQL)

#### Практика ⚒️

Разбор четырёх багов (разновидностей утечек памяти) на примере [лабораторного приложения](https://github.com/Toparvion/spring-petclinic-rest) Spring PetClinic REST по схеме:

1. Описание кейса
2. Имплементация (пояснения к реализации)
3. Проблема
4. Обнаружение (подсказки к проведению анализа)
5. Причина

Все три кейса последовательны: для воспроизведения следующего нужно будет найти и устранить причину предыдущего.

### Примеры слайдов
Из теоретической части занятия
{{< gallery album="training/memory" >}}

---

### Интересно?
Если хотите, чтобы я провёл этот тренинг в вашей компании, вы&nbsp;можете:
{{< cta cta_text="Оставить заявку" cta_link="https://forms.yandex.ru/u/66b620c5f47e7313456e26e4/" cta_new_tab="true" cta_alt_text="Задать вопрос" cta_alt_link="https://t.me/Toparvion" cta_alt_new_tab="true" >}}