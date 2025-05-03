---
title: "Путеводитель по профилированию приложений на JVM"
event: "Стачка 2025 (Ульяновск)"
event_url: https://ul25.nastachku.ru/%D0%BF%D1%83%D1%82%D0%B5%D0%B2%D0%BE%D0%B4%D0%B8%D1%82%D0%B5%D0%BB%D1%8C-%D0%BF%D0%BE-%D0%BF%D1%80%D0%BE%D1%84%D0%B8%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8E-%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B9-%D0%BD%D0%B0-jvm

location: Ульяновск, УлГПУ
address:
  street: Площадь Ленина 4
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
date: "2025-04-19T10:00:00Z"
date_end: "2025-04-19T10:40:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2025-04-18T00:00:00Z"

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
featured: false

image:
  caption: ''
  focal_point: Smart

links:
- name: Диплом спикера
  url: diploma.pdf
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
{{% callout note %}}
Это слегка сокращённая и уточнённая версия доклада, представленного на конференции [JPoint 2025](/event/2025/jpoint/).
{{% /callout %}}