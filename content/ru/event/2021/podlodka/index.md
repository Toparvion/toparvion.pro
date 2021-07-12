---
title: "Как внедрять тестовое поведение в чистовой код и спать спокойно?"
event: "Онлайн-конференция Podlodka Backend Crew, сезон #2"
event_url: https://podlodka.io/becrew

location: Онлайн
# address:
  # street: ул. Пирогова 1
  # city: Санкт-Петербург
  # region: CA
  # postcode: '630090'
  # country: Россия

summary: Доклад о том, как сохранять чистоту кода в сложных программных проектах
abstract: |
  Помнишь, как ты однажды случайно отправил на production кусочек тестового кода? А тот крохотный if, что по твоей задумке никогда не выполнится в боевой среде? А знаешь ли, сколько таких «закладок» болтается в промышленных приложениях и может выстрелить в любой момент? Много! В некоторых особо рисковых областях (например, в финтехе) борьба с их ростом превращается в отдельную задачу.

  О том, как выйти сухим из воды при добавлении тестового поведения в чистовой код, мы и поговорим в докладе:
  * разберём ситуации, требующие правок кода не для production;
  * какой арсенал лучше применить: «штатные» средства, аспектно-ориентированный подход или всё вместе;
  * как внедрять в приложение почти любое стороннее поведение, но при этом не пачкать репозиторий грязными хаками и даже не пересобирать само приложение.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2021-07-08T10:00:00+04:00"
date_end: "2021-07-08T11:00:00+04:00"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2021-07-02T00:00:00Z"

authors:
  - toparvion
tags:
  - тестирование
  - sei
  - side-effect-injection
  - aspectj
  - byteman
  - jmint
  - gluonj
  - аоп
  - байт-код

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
url_code: "https://github.com/Toparvion/jmint"
# url_pdf: ""
# url_slides: "https://speakerdeck.com/toparvion/iniektsiia-tiestovykh-poviedienii-kak-vyiti-sukhim-iz-vody"
url_video: "https://youtu.be/zzkKQztAzVM"

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
  - jmint

# Enable math on this page?
math: false
---
{{% callout note %}}
Это адаптированная для разработчиков версия [доклада](/event/2020/test-trend/) с&nbsp;онлайн митапа для тестировщиков TestTrend.
{{% /callout %}}
