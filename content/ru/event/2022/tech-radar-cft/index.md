---
title: "Экскурсия в бэкенд Интернета вещей"
event: "Конференция TechRadar (ЦФТ)"

location: Новосибирск, АкадемПарк
address:
  street: ул. Николаева 12/1
  country: Россия

summary: Обзорный доклад о некоторых задачах вокруг IoT глазами бэкендера
abstract: |
  Даже самый “умный” чайник на деле не так уж умён в отрыве от сервера, куда он передаёт свою температуру и откуда получает команды на подогрев. Но с ним хотя бы можно взаимодействовать напрямую. А как быть десяткам датчиков загрязнения воздуха на предприятии? Или сотням датчиков температуры на посевной площади? Или тысячам электронных замков в гостинице? Ясно, что бизнес-ценность таких устройств целиком раскрывается только на бэкенде.

  В этом докладе мы поверхностно пробежимся по некоторым задачам вокруг IoT и на примере одной интеграционной платформы на Java (с оглядкой на другие) посмотрим, как эти задачи могут решаться “на том конце провода”. Повникаем в визуализацию процессов и агрегатов, поговорим об интеллектуальном анализе машинных данных и разберёмся с тем, как унифицировать получение и обработку данных от огромного (зоо)парка всевозможных устройств.

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2022-05-20T18:20:00Z"
date_end: "2022-05-20T19:00:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2022-05-20T00:00:00Z"

authors:
  - toparvion
tags:
  - iot
  - java
  - iot-intro
  - cassandra
  - bacnet
  - lorawan
  - aggregate
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
#projects:
#  - no

# Enable math on this page?
math: false
---
{{% callout note %}}
Это слегка адаптированное повторение доклада с митапа [JUG.EKB #18](/event/2022/jugekb/).
{{% /callout %}}