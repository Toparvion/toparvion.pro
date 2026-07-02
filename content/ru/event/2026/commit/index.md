---
title: "Профилирование Java-приложений: подходы и инструменты"
event: "CommIT Meetup"
event_url: https://commit-meetup.tech/profilirovanie-java-prilozheniy-podkhody-i-instrumenty

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
date: "2026-07-02T17:00:00Z"
date_end: "2026-07-02T18:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2026-07-02T00:00:00Z"

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
Это слегка обновлённая версия доклада о&nbsp;профилировании, представленного на&nbsp;митапе [Podlodka Backend Crew](/event/2025/podlodka/).
{{% /callout %}}