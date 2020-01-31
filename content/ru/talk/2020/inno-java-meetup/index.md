---
title: "Грабли и плюшки обновления Spring Boot микросервисов с Java 8 на 11"
event: Inno Java Meetup
event_url: ""

location: Astana IT University, коворкинг
address:
  # street: Станционная, 104
  city: Нур-Султан
  # region: CA
  # postcode: '196140'
  country: Казахстан

summary: Доклад о проблемах и прелестях обновления JDK в микросервисах (сокращенный)
abstract: |
  Со своим новым циклом релизов Java балует нас версиями каждые полгода, но мало кто в enterprise-мире торопится на них переходить. Однако 11-ая версия Java стала исключением — благодаря сразу нескольким фишкам, многие захотели на нее перейти. И всё бы ничего, вот только если у вас парк микросервисов на Spring Boot, это обновление может стать чуть более «занимательным», чем просто перещёлкнуть версию...

  В докладе я расскажу не только и не столько о новых языковых фичах, сколько о граблях и плюшках на пути обновления Boot-микросервисов в целом: начиная со сборки (например, Gradle'ом) и заканчивая развёртыванием Docker-контейнеров (например, в Kubernetes). Попутно расскажу о том, чего ждать от перехода на Spring Boot версии 2.1 (начавшей поддерживать Java 11), а также о нескольких приятных JEP'ах, добавленных в последних версиях.


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2020-02-08T17:00:00Z"
date_end: "2019-02-08T17:40:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2020-01-31T00:00:00Z"

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
  caption: ''
  focal_point: Smart

# links:
# - icon: twitter
#   icon_pack: fab
#   name: Подписаться
#   url: https://twitter.com/georgecushen
url_code: "https://github.com/Toparvion/jpoint-2019-materials"
url_pdf: ""
url_slides: ""
url_video: ""

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
{{% alert note %}}
Это сокращенная версия [доклада](/talk/2019/jpoint/) на конференции JPoint'19.
{{% /alert %}}
