---
title: "Side Effect Injection или добродетельные костыли"
event: Конференция JBreak 2018
event_url: https://2018.jbreak.ru/talks/322rd0j4gaickgaymqi8gy/

location: Новосибирск, Экспоцентр
address:
  street: Станционная, 104
  # city: Санкт-Петербург
  # region: CA
  # postcode: '196140'
  country: Россия

summary: Доклад о подходе Side Effect Injection и реализующем его инструменте
abstract: |
  Помните тот случай, когда вы случайно (или нет) отправили в production кусочек кода, предназначенный только для теста? Или временно вставили крохотный if’чик, например, с `Thread.sleep()` или логированием для отладки? Знайте, вы не одиноки. Есть куча реальных задач, после решения которых в production частенько уезжает тестовый/отладочный код, превращаясь там в бомбу замедленного действия, попутно преумножая тех.долг и пятно на карме разработчика.

  В докладе мы по косточкам разберем подход Side Effect Injection, который позволит вам внедрять в тестируемое приложение почти любое поведение: задержки, заглушки, логирование, обход безопасности и т.д., но при этом не пачкать репозиторий грязными хаками и даже не пересобирать само приложение.

  В ходе разбора полюбуемся вариантами компиляции Java-кода, расковыряем один кейс модификации байт-кода в JVM, препарируем формальную грамматику Java, а затем поиграем со всем этим на примере реального приложения.


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2018-03-04T13:45:00Z"
date_end: "2018-03-04T14:45:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2018-02-01T00:00:00Z"

authors:
  - toparvion
tags:
  - sei
  - jmint
  - отладка
  - тестирование
  - на-русском
  - байткод
  - инструментирование
  - java-агент

# Is this a featured talk? (true/false)
featured: true

image:
  caption: 'Во время доклада'
  focal_point: Smart

# links:
# - icon: twitter
#   icon_pack: fab
#   name: Подписаться
#   url: https://twitter.com/georgecushen
url_code: "https://github.com/Toparvion/jmint"
url_pdf: ""   # локальный файл будет обнаружен автоматически
url_slides: ""
url_video: "https://youtu.be/5gEyr-25rfE"

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
