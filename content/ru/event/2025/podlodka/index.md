---
title: "Профилирование Java-приложений: подходы и инструменты"
event: "Podlodka Java Crew"
event_url: https://podlodka.io/javacrew

location: онлайн
#address:
#  street: Краснопресненская наб., 12
#  country: Россия

summary: Базовый конспект инженера по профилированию программ на JVM
abstract: |
  В этом докладе-конспекте разберемся с подходами к профилированию, выясним их слабые и сильные стороны, поймём области применения.
  А затем, опираясь на полученные знания, рассмотрим несколько популярных профайлеров, реализующих эти подходы и позволяющих анализировать производительность в самых разных ситуациях.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2025-05-27T14:00:00Z"
date_end: "2025-05-27T15:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2025-05-03T00:00:00Z"

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
  - performance
  - на-русском
  - онлайн

# Is this a featured talk? (true/false)
featured: false

image:
  caption: ''
  focal_point: Smart

links:
# - icon: twitter
#   icon_pack: fab
#   name: Подписаться
#   url: https://twitter.com/georgecushen
url_code: ""
url_pdf: ""
url_slides: ""
url_video: "https://youtu.be/TAts7OI5hl8?si=_j9pygtn0FwQpROL"

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
Это слегка переработанная версия доклада о&nbsp;профилировании, представленного на&nbsp;конференции [JPoint&nbsp;2025](/event/2025/jpoint/).
{{% /callout %}}