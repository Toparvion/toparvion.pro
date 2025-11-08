---
title: "Анализ потоков"
subtitle: "Тренинг по работе с дампами потоков"
summary: "Тренинг по работе с дампами потоков"
authors:
  - toparvion
tags:
  - jvm
  - threads
  - dump
  - monitoring
  - jstack
  - deadlock
categories:
  - Performance
trainings:
  - performance
date: 2024-06-30
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
- album: examples/training/threads
  image: sample-slide-1.png
  caption: Как снять дамп потоков без спец.инструментов
- album: examples/training/threads
  image: sample-slide-2.png
  caption: Фрагмент обхода элементов дампа
- album: examples/training/threads
  image: sample-slide-3.png
  caption: Синтаксис запуска утилиты jattach
- album: examples/training/threads
  image: sample-slide-4.png
  caption: Размеченный скриншот инструмента VisualVM

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
### Описание

Примерно каждый первый профессиональный разработчик на Java, Kotlin и других JVM языках рано или поздно сталкивается с проблемами производительности, связанными с многопоточностью. Начиная от банального залипания одного потока и заканчивая изощрёнными многоступенчатыми deadlock’ами, такие проблемы могут докучать отдельным “счастливым” пользователям или вовсе положить весь production. И полностью предотвратить их практически невозможно.

Основным инструментом диагностики в этих случаях являются **дампы потоков** (thread dumps). Но чтобы они действительно помогали, нужно знать, какими способами их получать в разных ситуациях, чем анализировать в зависимости от проблемы и, конечно, как по ним находить и устранять причины сбоев.

Этим задачам и посвящён данный тренинг. Помимо них, вы также научитесь выявлять разницу в поведении потоков через дампы и понимать, в каких случаях дампы потоков можно и нужно заменить/дополнить другими инструментам диагностики.

{{% callout note %}}
Формат и стоимость тренинга приведены на странице серии тренингов про [Производительность](/training/performance/).
{{% /callout %}}

### Содержание

#### Теория 🎓

1. Простое определение дампа потоков

2. Состав данных в дампе

3. Показания и противопоказания к применению

4. Занятость потока и ее связь с состояниями

5. Инструменты для снятия дампа потоков (6 видов)

6. Схожие функции в родственных инструментах (6 примеров)

7. Инструменты для анализа дампов (5 шт.)

8. Приёмы и техники анализа:

   * Учёт состояний потоков
   * Навигация по анонимным классам
   * Выявление изменений во времени
   * Что нужно знать о пулах потоков

#### Практика ⚒️

Разбор четырёх performance-багов, искусственно привнесенных в [тестовое приложение](https://github.com/Toparvion/spring-petclinic-rest) Spring PetClinic REST.

Все четыре бага (учебных кейса) являются разновидностями блокировок &mdash; от почти очевидных до распределённых (с участием СУБД). Все они подробно описаны по следующей схеме:

1. Легенда (краткое описание бизнес-логики кейса)
2. Техническая реализация (краткое описание имплементации)
3. Проблема
4. Шаги воспроизведения
5. Объяснение (разгадка проблемы с комментариями)
6. Варианты решения (чаще всего несколько)

{{% callout note %}}
Кейс №4 требует подключения к PostgreSQL для воспроизведения распределённого deadlock’а. Для простоты запуска в репозитории проекта предусмотрен Docker Compose файл, но можно развернуть и вручную.
{{% /callout %}}

### Примеры слайдов
Из теоретической части занятия
{{< gallery album="training/threads" >}}

### Материалы в шпаргалке

* Когда нужен дамп потоков
* Чем снимать
* Чем открывать
* Что искать сначала

---

### Интересно?
Если хотите, чтобы я провёл этот тренинг в вашей компании, вы&nbsp;можете:
{{< cta cta_text="Оставить заявку" cta_link="https://forms.yandex.ru/u/66b620c5f47e7313456e26e4/" cta_new_tab="true" cta_alt_text="Задать вопрос" cta_alt_link="https://t.me/Toparvion" cta_alt_new_tab="true" >}}