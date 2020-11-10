---
title: "Инъекция тестовых поведений: как выйти сухим из воды?"
event: Java Meetup Innopolis
event_url: "https://innopolis.timepad.ru/event/1382976/"

location: г. Иннополис (онлайн)
# address:
  # street: ул. Туркистан
  # city: Нур-Султан
  # region: CA
  # postcode: '196140'
  # country: Казахстан

summary: Доклад о том, как внедрять тестовое поведение в чистовой код Java приложений и спать спокойно
abstract: |
  Помнишь, как ты однажды случайно отправил на production кусочек кода, предназначенный только для теста? А тот крохотный if, что по твоей задумке никогда не выполнится в боевой среде? А знаешь ли, сколько таких «закладок» болтается в промышленных приложениях и может выстрелить в любой момент? Много! В некоторых особо рисковых областях (например, в финтехе) борьба с их ростом превращается в отдельную задачу.

  О том, как добавлять в чистовой код тестовое поведение и спать спокойно, мы и поговорим в докладе:  
  * разберём ситуации, требующие правок кода не для production  
  * какой арсенал лучше применить: «штатные» средства, аспектно-ориентированный подход или всё вместе  
  * как внедрять в приложение почти любое тестовое и отладочное поведение, но при этом не пачкать репозиторий грязными хаками и даже не пересобирать само приложение.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2020-07-30T17:00:00Z"
date_end: "2020-07-30T18:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2020-07-24T16:00:00Z"

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
url_slides: "https://speakerdeck.com/toparvion/iniektsiia-tiestovykh-poviedienii-kak-vyiti-sukhim-iz-vody"
# url_video: ""

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
Это полностью переработанная версия [доклада](/talk/2018/jbreak/) с&nbsp;конференции JBreak'18.
{{% /callout %}}
