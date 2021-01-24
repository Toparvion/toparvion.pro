---
title: "Как внедрять тестовое поведение в чистовой код?"
event: "TestTrend Online meetup"
event_url: https://team.cft.ru/events/181

location: Новосибирск (онлайн)
# address:
  # street: ул. Пирогова 1
  # city: Санкт-Петербург
  # region: CA
  # postcode: '630090'
  # country: Россия

summary: Получасовая сессия "лампового хардкора" для тестировщиков и им сочувствующих
abstract: |
  Как же порой трудно добиться нужного поведения программы в тестовом окружении: фейковые данные не соответствуют реальности, корявая внешняя зависимость не даёт воспроизвести проблемный кейс, а вездесущая безопасность постоянно ставит палки в колёса… И хотя для всех этих проблем есть какие-то внешние решения, в идеале все такие проблемы можно было бы устранить прямо внутри приложения. Но разве можно лезть внутрь “черного ящика”? Можно! Если осторожно :wink:

  О том, как добавлять в чистовой код тестовое поведение и спать спокойно, мы и поговорим в докладе:

  * посмотрим на разные жизненные ситуации, требующие влезания в “черный ящик”;

  * узнаем, как ещё можно бороться с диагнозом “у меня не воспроизводится”, когда больше ничего не помогает;

  * разберёмся, как внедрять в приложение почти любое тестовое и отладочное поведение, но при этом не пачкать репозиторий грязными хаками и даже не пересобирать само приложение.

  Доклад ориентирован, в первую очередь, на тестировщиков и разработчиков серверных и настольных приложений на Java, но может быть интересен и другим специалистам.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2020-12-02T19:15:00Z"
date_end: "2020-12-02T20:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2020-11-27T00:00:00Z"

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
url_slides: "https://speakerdeck.com/toparvion/kak-vniedriat-tiestovoie-poviedieniie-v-chistovoi-kod"
url_video: "https://youtu.be/rCHJ7iQr6To?t=874"

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
Это адаптированное повторение [доклада](/talk/2020/inno-java-meetup-online/) с&nbsp;онлайн-митапа по&nbsp;Java в&nbsp;Иннополисе.
{{% /callout %}}
