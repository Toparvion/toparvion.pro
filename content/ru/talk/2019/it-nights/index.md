---
title: Грабли и плюшки обновления Spring Boot микросервисов с Java 8 на 11
event: Конференция IT Nights 2019
event_url: https://it-nights.ru/

location: Иннополис (Татарстан)
address:
  # street: Петербургское шоссе, 64/1
  # city: Санкт-Петербург
  # region: CA
  # postcode: '196140'
  country: Россия

summary: Иммерсивный доклад в библиотеке Университета Иннополиса
abstract: |
  Со своим новым циклом релизов Java балует нас версиями каждые полгода, но мало кто в enterprise-мире торопится на них переходить. Однако 11-ая версия Java стала исключением — благодаря сразу нескольким фишкам, многие захотели на нее перейти. И всё бы ничего, вот только если у вас парк микросервисов на Spring Boot, это обновление может стать чуть более «занимательным», чем просто перещёлкнуть версию…

  В докладе я расскажу не только и не столько о новых языковых фичах, сколько о граблях и плюшках на пути обновления Boot-микросервисов в целом: начиная со сборки (например, Gradle'ом) и заканчивая развёртыванием Docker-контейнеров (например, в Kubernetes). Попутно расскажу о том, чего ждать от перехода на Spring Boot версии 2.1 (начавшей поддерживать Java 11), а также о нескольких приятных JEP'ах, на рассмотрении которых можно будет остановиться поподробнее или даже увидеть их в действии.


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2019-08-01T20:50:00Z"
date_end: "2019-08-01T21:50:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2019-07-01T00:00:00Z"

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
url_pdf: ""
url_slides: "https://speakerdeck.com/toparvion/pierievod-spring-boot-mikrosiervisov-s-java-8-na-11-chto-mozhiet-poiti-nie-tak"
url_video: "https://www.youtube.com/playlist?list=PLgZXqi5nH1m2EfbrLLqnvg9rY1doSi4ko"

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
