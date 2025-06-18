---
title: "Как сделать исполнение low-code прозрачным: опыт большой IoT-платформы"
event: "Saint HighLoad++"
event_url: https://highload.ru/spb/2025/abstracts/15366

location:  Санкт-Петербург, DESIGN DISTRICT DAA
address:
  street: Красногвардейская площадь, 3Е
  country: Россия

summary: Сводка опыта в построении сложной подсистемы нагруженного приложения
abstract: |
  Low-code- и no-code-решения привлекают своей простотой, вроде как устраняющей нужду в отладке, трассировке, профилировании и других инструментах «классического» программирования. Но в масштабах реальных промышленных проектов без них оказывается трудно, ведь иначе как понять, что делает та или иная low-code-конструкция под капотом? И, что особенно важно, как она ведет себя под нагрузкой?

  В докладе мы посмотрим на имплементацию этих функций на примере российской IoT-платформы AggreGate, где встроенный язык выражений был наделен средствами трассировки и визуализации, позволяющими low-code-разработчикам видеть ход и результаты выполнения их команд на всех этапах от редактора до production. Особый акцент сделаем на производительности:
  построение/слияние/кэширование деревьев разбора, ленивая загрузка результатов, троттлинг вычислений — словом, все, что позволяет сохранить сервер живым, а отладку — пригодной даже при сотнях тысяч RPS.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2025-06-24T14:00:00Z"
date_end: "2025-06-24T15:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2025-06-18T00:00:00Z"

authors:
  - toparvion
tags:
  - highload
  - patterns
  - caching
  - architecture
  - language
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
