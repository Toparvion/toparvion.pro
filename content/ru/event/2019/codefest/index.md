---
title: "Перевод Spring Boot-микросервисов с Java 8 на 11: что может пойти не так?"
event: Конференция CodeFest X
event_url: https://2019.codefest.ru/lecture/1433

location: Новосибирск, Экспоцентр
address:
  street: Станционная, 104
  # city: Санкт-Петербург
  # region: CA
  # postcode: '196140'
  country: Россия

summary: Доклад о проблемах и прелестях обновления JDK в микросервисах (сокращенный)
abstract: |
  С новым циклом релизов платформа Java стала баловать нас версиями каждые полгода, но мало кто в enterprise-мире торопится на них переходить. Однако Java 11 стала исключением — благодаря сразу нескольким факторам, она показалась многим подходящей целью для обновления. И всё бы ничего, но если у вас парк микросервисов на Spring Boot, это обновление может стать несколько более «занимательным», чем просто перещёлкнуть версию...

  В докладе речь пойдет не только и не столько о новых языковых фичах, сколько о граблях на пути обновления Boot-микросервисов в целом: начиная со сборки (например, Gradle’ом) и заканчивая развёртыванием Docker-контейнеров (например, в Kubernetes). Отдельно поговорим о том, чего ждать от перехода на Spring Boot версии 2.1 (начавшей поддерживать Java 11) и его спутников, привносящих не мало новшеств и «спецэффектов».


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2019-03-31T12:00:00Z"
date_end: "2019-03-31T12:40:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2019-02-15T00:00:00Z"

authors:
  - toparvion
tags:
  - микросервисы
  - spring-boot
  - на-русском
  - проект-восход  
  - грабли
  - gradle
  - jshell
  - lombok
  - jdk11
  - shebang-java

# Is this a featured talk? (true/false)
featured: false

image:
  caption: 'Перед радио эфиром'
  focal_point: Smart

# links:
# - icon: twitter
#   icon_pack: fab
#   name: Подписаться
#   url: https://twitter.com/georgecushen
url_code: "https://github.com/Toparvion/jpoint-2019-materials"
url_pdf: ""
url_slides: "https://speakerdeck.com/codefest/codefest-2019-vladimir-plizgha-tsft-pierievod-spring-boot-mikrosiervisov-s-java-8-na-11-chto-mozhiet-poiti-nie-tak"
url_video: "https://youtu.be/PmoAyrjX22I"

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
{{% callout note %}}
Это сокращенная версия [доклада](/talk/2019/jpoint/) на конференции JPoint'19.
{{% /callout %}}
