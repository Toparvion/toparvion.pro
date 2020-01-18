---
title: "Сборка чугунного скорохода на Spring Boot и AppCDS"
authors:
- toparvion
date: "2019-10-14T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2019-10-23T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["0"]

# Publication name and optional abbreviated publication name.
publication: "[На Хабре](https://habr.com/ru/post/472638/)"
publication_short: ""

abstract: |
  **Application Class Data Sharing (AppCDS)** – фича JVM для ускорения запуска и экономии памяти. Появившись в HotSpot в зачаточном виде ещё в JDK 1.5 (2004 г.), она долгое время оставалась весьма ограниченной, да ещё и отчасти коммерческой. Лишь только с OpenJDK 10 (2018 г.) её сделали доступной простым смертным, заодно расширив область применения. А недавно вышедшая Java 13 попыталась сделать это применение более простым.

  Идея AppCDS в том, чтобы “расшарить” однажды прогруженные классы между экземплярами одной и той же JVM на одном хосте. Кажется, это должно здорово зайти микросервисам, особенно “бройлерам” на Spring Boot с их тысячами библиотечных классов, ведь теперь эти классы не надо будет загружать (парсить и верифицировать) при каждом старте каждого инстанса JVM, и они не будут дублироваться в памяти. А значит, запуск должен стать скорее, а потребление памяти – ниже. Чудно, не правда ли?

  Всё так, всё так. Но если ты, однохабрянин, привык верить не бульварным вывескам, а конкретным цифрам и примерам, то добро пожаловать под кат – попробуем разобраться, как оно на самом деле…

# Summary. An optional shortened abstract.
summary: Результаты исследования применимости AppCDS к Spring Boot

tags:
- spring-boot
- на-русском
- appcds
- производительность
featured: false

links:
- name: "Хабр"
  url: "https://habr.com/ru/post/472638/"
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
  caption: "*Photo by [Nick Fewings](https://unsplash.com/@jannerboy62?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/locomotive-building?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*"
  focal_point: Smart
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects:
- jcudos

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
# slides: example
---
{{% alert note %}}
Вскоре после выхода, чтобы улучшить оформление, эта статья была повторно [опубликована](/post/2019/10/appcds-with-spring-boot/) на настоящем сайте под другим названием.
{{% /alert %}}
