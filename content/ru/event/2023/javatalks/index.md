---
title: "Скажите «Ой»: JVM и OOM Killer"
event: "CFT JavaTalks Meetup"
event_url: https://team.cft.ru/events/225

location: Новосибирск, бар Red Rabbit Академгородок
address:
  street: ул. Ильича 8
  country: Россия

summary: Взгляд на проблемы с нативной памятью JVM глазами прикладного разработчика (сокращённая версия)
abstract: |
  Когда слышишь об `OutOfMemory` в Java, на ум приходят всякие раздувшиеся коллекции, забытый `finally` или тонны бесполезных объектов, почему-то зависших в куче. Для устранения таких причин есть стектрейсы, хип-дампы и другие полезные штуки, но главное – возможность исправить что-то в своём коде.

  Но как быть, если нехватка случилась в нативной памяти, вне кучи? Как определить ее источник? На что можно повлиять, если напрямую с этим не работаешь? Почему вообще такое может произойти, и что делать, чтобы этого избежать?

  В докладе посмотрим на эти вопросы глазами прикладного разработчика, прощупаем JVM **Native Memory Tracking** (и его недостатки), заглянем в устройство oomKiller’а в Linux и познакомимся с инструментами, которые бы уж лучше никогда не пригождались…

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2023-11-23T20:45:00Z"
date_end: "2023-11-23T21:20:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2022-11-30T00:00:00Z"

authors:
  - toparvion
tags:
  - jvm
  - native memory
  - nmt
  - async-profiler
  - linux
  - outofmemory
  - oom
  - jemalloc
  - gc
  - jcmd
  - на-русском

# Is this a featured talk? (true/false)
featured: false

image:
  caption: '© Сергей Колесников'
  focal_point: Smart

#links:
# - name: Фото
#   url: "https://phkolesnikov.ru/disk/23-11-2023-tsft-86qxrq"
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
#projects:
#  - no

# Enable math on this page?
math: false
---
{{% callout note %}}
Это сокращённая версия доклада с митапа [JUGNsk Meetup #21](/event/2023/jugnsk/).
{{% /callout %}}