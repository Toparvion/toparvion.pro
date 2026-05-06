---
title: "Анализируем heap-дампы с прода, не привлекая внимания безопасников"
authors:
- toparvion
date: "2026-04-12T10:00:00+00:00"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2026-05-06T19:48:00+07:00"

# Publication type.
# Legend: https://docs.citationstyles.org/en/stable/specification.html#appendix-iii-types
publication_types: ["post"]

# Publication name and optional abbreviated publication name.
publication: "[Блог компании Spring АйО](https://habr.com/ru/companies/spring_aio/articles/1031462/)"
publication_short: ""

abstract: |
  Heap-дампы JVM – бесценный источник информации при разборе аварий с OutOfMemory и оптимизации производительности. Но вместе с тем они же – потенциальные каналы утечки данных, ведь будучи снятыми с боевого сервиса, дампы уносят в себе всё, с чем работал сервис на момент снимка: логины, пароли (иногда в открытом виде), важные ID и т.п. – словом, всяческие sensitive данные, которые не нужны для анализа, но которые навлекают на получателя дампа серьёзную ответственность и риски. Как этого избежать без ущерба делу – разбираемся в этой статье.
# Summary. An optional shortened abstract.
summary: Краткий обзор инструментов для обфускации heap-дампов

tags:
  - java
  - hprof
  - dump
  - mat
  - на-русском
featured: false

links:
- name: "Хабр"
  url: "https://habr.com/ru/companies/spring_aio/articles/1031462/"
- name: "Telegram"
  url: "https://t.me/spring_aio/1056"
url_pdf: ''
url_code: ''
url_dataset: ''
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
image:
  caption: ""
  focal_point: Smart
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
# slides: example
---

