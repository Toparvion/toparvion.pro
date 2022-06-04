---
title: "Перевод Spring Boot-микросервисов с Java 8 на 11: что может пойти не так?"
event: Конференция JPoint 2019
event_url: https://2019.jpoint.ru/talks/6kyixr30fgfmedztyom628/

location: Москва, ЦМТ
address:
  street: Краснопресненская набережная, 12
  # city: Санкт-Петербург
  # region: CA
  # postcode: '196140'
  country: Россия

summary: Доклад о проблемах и прелестях обновления JDK в микросервисах
abstract: |
  С новым циклом релизов платформа Java стала баловать нас версиями каждые полгода, но мало кто в enterprise-мире торопится на них переходить. Однако Java 11 стала исключением — благодаря сразу нескольким факторам она показалась многим подходящей целью для обновления. И всё бы ничего, но если у вас парк микросервисов на Spring Boot, это обновление может стать несколько более занимательным, чем просто перещёлкнуть версию...

  В докладе речь пойдет не только и не столько о новых языковых фичах, сколько о граблях на пути обновления Boot-микросервисов в целом, начиная со сборки (например, Gradle-ом) и заканчивая развёртыванием Docker-контейнеров (например, в Kubernetes). Отдельно поговорим о том, чего ждать от перехода на Spring Boot версии 2.1 (начавшей поддерживать Java 11) и его спутников, привносящих немало новшеств и спецэффектов.

  Доклад ориентирован на тех разработчиков и тестировщиков микросервисных приложений на Spring Boot, которые переводят или планируют перевести свои продукты на 11-ю версию платформы Java.


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2019-04-06T16:15:00Z"
date_end: "2019-04-06T17:15:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2019-03-01T00:00:00Z"

authors:
  - toparvion
tags:
  - микросервисы
  - spring-boot
  - на-русском
  - проект-восход  
  - грабли
  - gradle
  - appcds
  - jshell
  - lombok
  - jdk11
  - shebang-java

# Is this a featured talk? (true/false)
featured: false

image:
  caption: 'Во время доклада'
  focal_point: Smart

# links:
# - icon: twitter
#   icon_pack: fab
#   name: Подписаться
#   url: https://twitter.com/georgecushen
url_code: "https://github.com/Toparvion/jpoint-2019-materials"
url_pdf: ""   # локальный файл будет обнаружен автоматически
url_slides: "https://speakerdeck.com/toparvion/pierievod-spring-boot-mikrosiervisov-s-java-8-na-11-chto-mozhiet-poiti-nie-tak"
url_video: "https://youtu.be/fmLW7VkSuN8?list=PLVe-2wcL84b94ehIDEskAMbQKDzt5JZNS"

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
projects:
  - jcudos

# Enable math on this page?
math: false
---
