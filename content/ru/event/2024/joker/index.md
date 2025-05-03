---
title: "Путеводитель по анализу памяти JVM-приложений"
event: "Joker 2024"
event_url: https://jokerconf.com/talks/73c199ab870d45f9b27439962cd77f91/

location: Санкт-Петербург, Design District DAA
address:
  street: Красногвардейская пл., 3
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
date: "2024-10-15T15:30:00Z"
date_end: "2024-10-15T16:15:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2022-10-14T00:00:00Z"

authors:
  - toparvion
tags:
  - jvm
  - outofmemory
  - oom
  - gc
  - mat
  - visualvm
  - на-русском

# Is this a featured talk? (true/false)
featured: true

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
url_slides: ""
url_video: "https://vkvideo.ru/video-796_456240545"

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
  - heap-talk

# Enable math on this page?
math: false
---

{{% callout note %}}
Это сокращённая и обновлённая версия доклада, представленного на митапе [JUGNsk #24](/event/2024/jugnsk/).
{{% /callout %}}