---
title: "Spring Boot 2: чего не пишут в release notes"
event: Конференция Joker 2018
event_url: https://2018.jokerconf.com/2018/talks/2ixtotkht6eksa6guii6yq/

location: Санкт-Петербург, Экспофорум
address:
  street: Петербургское шоссе, 64/1
  # city: Санкт-Петербург
  # region: CA
  postcode: '196140'
  country: Россия

summary: Доклад о сложностях обновления базового фреймворка микросервисов
abstract: |
  Обновляя у себя какой-либо фреймворк, ты, конечно, всегда внимательно читаешь его release notes и migration guide:wink: Но даже если это правда, тебя может поджидать множество сюрпризов, особенно если это мажорное обновление такого базового фреймворка, как Spring Boot. Помимо себя, он привносит обновления для своего BOM, а это &approx;150 транзитивных зависимостей на все случаи жизни — такой upgrade не может пройти без накладок...

  В этом докладе («грабледайджесте») я расскажу о своем опыте перевода микросервисного приложения на Spring Boot 2, проведу по многим собранным в ходе этого граблям и покажу для каждого случая решение или обходной путь. В частности, я расскажу:

  - как поддержка реактивного стека ломает обратную совместимость на уровне исходного кода;
  - почему переписанный с нуля Gradle-плагин может портить сборку составного - проекта и содержимое выходного JAR;
  - как новые правила relax binding параметров могут помешать запуску приложения;
  - как сломать старт микросервиса, ненароком поссорив мониторинг с пулом коннектов;
  - причем тут движок Micrometer, если отвалились задания по расписанию;
  - как замешан новый режим проксирования в исчезновении некоторых MBean из JMX;
  - как рефакторинг настроек WebMVC может своротить определение Content-Type при отдаче файлов;
  - какие новшества Mockito не дают тестам скомпилироваться и проваливают надежно проходившие тесты;
  - а также о ряде других мелких, но неприятных «спецэффектов».

  Доклад ориентирован на разработчиков, планирующих или уже внедряющих Spring Boot 2 поверх старой версии или с нуля.


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2018-10-19T16:00:00Z"
date_end: "2018-10-19T17:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2018-09-01T00:00:00Z"

authors:
  - toparvion
tags:
  - микросервисы
  - spring-boot
  - на-русском
  - грабли

# Is this a featured talk? (true/false)
featured: true

image:
  caption: 'Во время доклада'
  focal_point: Smart

links:
- name: Текст
  url: https://habr.com/ru/company/jugru/blog/439796/
url_code: "https://github.com/Toparvion/joker-2018-samples"
url_pdf: ""   # локальный файл будет обнаружен автоматически
url_slides: "https://speakerdeck.com/toparvion/spring-boot-2-chiegho-nie-pishut-v-release-notes"
url_video: "https://youtu.be/8jNXZXdb3no"

# Markdown Slides (optional).
#   Associate this talk with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []

# Enable math on this page?
math: false
---
