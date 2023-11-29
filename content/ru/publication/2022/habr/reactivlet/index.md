---
title: "Уж+ёж: реактивные компоненты в сервлетном окружении"
authors:
- toparvion
date: "2021-12-16T19:50:00+07:00"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2022-01-25T09:55:00+07:00"

# Publication type.
# Legend: https://docs.citationstyles.org/en/stable/specification.html#appendix-iii-types
publication_types: ["post"]

# Publication name and optional abbreviated publication name.
publication: "[Блог ЦФТ на Хабре](https://habr.com/ru/company/cft/blog/647479/)"
publication_short: ""

abstract: |
  Когда рассказывают о прелестях реактивного фреймворка [Spring WebFlux](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux) и его подкапотном [Project Reactor](https://projectreactor.io/), для примера чаще всего показывают новые, создаваемые с нуля приложения. Однако на практике приходится строить из готовых блоков, в том числе собственных прикладных и инфраструктурных модулей, которые уже написаны в императивном стиле и опираются на сервлетный стек. Как правило, такие модули нельзя/некогда/неохота *(нужное подчеркнуть)* переписывать, поэтому надо как-то адаптировать их создаваемому реактивному приложению с минимумом правок (а лучше без них вовсе). О некоторых подходах к такой задаче и пойдёт речь в этой серии из 3 заметок.

# Summary. An optional shortened abstract.
summary: Серия заметок о плавном внедрении реактивного подхода в сервлетные enterprise-приложения на Spring

tags:
  - spring-weblux
  - spring-webmvc
  - project-reactor
  - logging
  - mdc
  - gradle
  - servlet-api
  - openfeign
  - migration
featured: false

links:
- name: "Хабр"
  url: "https://habr.com/ru/company/cft/blog/647479/"
url_pdf: ''
url_code: "https://github.com/Toparvion/reactivlet-sample"
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
{{% callout note %}}
Эта серия статей была изначально опубликована [дома](/series/reactivlet/).
{{% /callout %}}
