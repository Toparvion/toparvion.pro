---
title: "Путеводитель по профилированию приложений на JVM"
event: "JPoint 2025"
event_url: https://jpoint.ru/talks/8f6a60900d8e488f98b81333137a093e/

location: Москва, Конгресс-центр ЦМТ
address:
  street: Краснопресненская наб., 12
  country: Россия

summary: Базовый конспект инженера по профилированию программ на JVM
abstract: |
  Когда ваше приложение тормозит, да еще на production, важно уметь быстро ответить на два вопроса: где узкое место и почему оно тормозит? Для этого, как правило, применяют профилирование. Вот только инструментов для него — зоопарк, и отличаются они чуть ли не до противоположности:

  - бесплатные и платные;
  - встроенные и внешние;
  - CLI и GUI.
  
  При этом все они используются, а значит, каждый в чем-то хорош. Но какой и в чем?

  С этим мы и разберемся во время доклада. Рассмотрим особенности профилирования, из-за которых появились разные инструменты, а затем на примере трех популярных профайлеров выясним, в чем их сильные и слабые стороны, в каких случаях выбирать тот или иной и как выжать из каждого максимум. Попутно пройдемся по граблям, на которые можно наступить, выбрав не тот инструмент или применив его неверно.

  Будет полезно разработчикам, ответственным не только за написание кода, но и за его производительность «в бою», а также инженерам по мониторингу и работе с инцидентами на production.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2025-04-04T15:45:00Z"
date_end: "2025-04-04T16:40:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2025-04-02T00:00:00Z"

authors:
  - toparvion
tags:
  - jvm
  - profiling
  - sampling
  - tracing
  - cpu
  - async-profiler
  - yourkit
  - jfr
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
url_code: ""
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
