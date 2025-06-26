---
title: "Путеводитель по анализу памяти JVM"
event: "JUGNsk Meetup #24"
event_url: https://jugnsk.timepad.ru/event/3036878/

location: Новосибирск, АкадемПарк (ЦКП)
address:
  street: ул. Николаева 12
  country: Россия

summary: Базовый конспект инженера по производительности программ на JVM
abstract: |
  Проблемы производительности JVM часто заставляют ковыряться в памяти приложения: то общие метрики надо посмотреть, а то и залезть в самую глубь за тем самым байтом. И хорошо бы знать заранее, где, что и как можно найти, а не судорожно гуглить и перебирать варианты, когда на production уже пригорело…

  В докладе мы посмотрим на некоторые типичные проблемы с памятью приложений на HotSpot JVM и подходящие им способы анализа:

  - вручную и полуавтоматически;
  - на горячую и постмортем;
  - встроенными средствами JVM/JDK и сторонними инструментами.

  Доклад будет полезен разработчикам, ответственным не только за написание кода, но и за его производительность "в бою", а также инженерам по мониторингу и работе с инцидентами на production.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2024-09-26T20:00:00Z"
date_end: "2024-09-26T21:05:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2022-09-26T00:00:00Z"

authors:
  - toparvion
tags:
  - jvm
  - outofmemory
  - oom
  - gc
  - mat
  - performance
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
url_code: "https://github.com/Toparvion/spring-petclinic-rest"
url_pdf: ""
url_slides: "https://speakerdeck.com/toparvion/putievoditiel-po-analizu-pamiati-jvm"
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
#projects:
#  - no

# Enable math on this page?
math: false
---

{{% callout note %}}
Этот доклад был позднее представлен в сокращённом и обновлённом виде на конференции [Joker 2024](/event/2024/joker/).
{{% /callout %}}