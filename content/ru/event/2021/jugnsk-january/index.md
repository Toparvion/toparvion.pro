---
title: "Spring Boot «fat» JAR: Тонкие части толстого артефакта"
event: "JUGNsk Meetup #17 (online)"
event_url: https://www.meetup.com/ru-RU/JUGNsk/events/275807033/

location: Новосибирск (онлайн)
# address:
  # street: ул. Пирогова 1
  # city: Санкт-Петербург
  # region: CA
  # postcode: '630090'
  # country: Россия

summary: Сводка заметок и советов по работе с "толстыми" архивами Spring Boot
abstract: |
  Одна из известнейших фич Spring Boot — упаковка целого приложения в т.н. «толстый» JAR, который потом «just runs». Это реально работает, и для многих ситуаций этого достаточно. Но если вы не доверяете магии и/или столкнулись с проблемами при развертывании «толстого» JAR, то вам пора вникнуть в устройство этого механизма.

  И тут выясняется, что «just runs» обходится далеко не бесплатно: есть ограничения по загрузке классов, вопросы к скорости запуска, конфликты со встроенными утилитами JDK, отличия в режимах dev/test/prod, а в некоторых случаях применение этой фичи и вовсе излишне.

  В этих и других тонкостях «толстого» JAR мы и разберемся в докладе. Заглянем в его устройство и поймём, в каких случаях он хорош, а в каких лучше обойтись без него (и что тогда выбрать вместо). Особое внимание уделим развертыванию в контейнерах.

  Доклад рассчитан на практикующих инженеров, поставляющих приложения на Spring Boot в production.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2021-01-28T20:00:00Z"
date_end: "2021-01-28T21:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2021-01-26T00:00:00Z"

authors:
  - toparvion
tags:
  - микросервисы
  - spring-boot
  - fat-jar
  - на-русском

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
url_code: "https://github.com/Toparvion/fat-jar-sample"
url_pdf: ""
url_slides: "https://speakerdeck.com/toparvion/spring-boot-fat-jar-tonkiie-chasti-tolstogho-artiefakta"
url_video: "https://youtu.be/ib_yvGhuzDg?t=5095"

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
{{% callout note %}}
Это повторение [доклада](/event/2020/joker/) с&nbsp;онлайн конференции Joker 2020.
{{% /callout %}}
