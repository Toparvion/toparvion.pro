---
title: "Дешифрация TLS трафика Java приложений с помощью логов"
authors:
- toparvion
date: "2015-03-27T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2015-03-27T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["0"]

# Publication name and optional abbreviated publication name.
publication: "[На Хабре](https://habr.com/ru/post/254205/)"
publication_short: ""

abstract: |
  Отладка защищенных по SSL/TLS интеграций у Java приложений порой становится весьма нетривиальной задачей: соединение не ставится/рвется, а прикладные логи могут оказаться скудными, доступа к правке исходных кодов может не быть, перехват трафика Wireshark'ом и попытка дешифрации приватным ключом сервера (даже если он есть) может провалиться, если в канале применялся шифр с PFS; прокси-сервер вроде Fiddler или Burp может не подойти, так как приложение не умеет ходить через прокси или на отрез отказывается верить подсунутому ему сертификату…

  Недавно на Хабре появилась [публикация](http://habrahabr.ru/post/253521/) от ValdikSS о том, как можно с помощью Wireshark расшифровать любой трафик от браузеров Firefox и Chrome без обладания приватным ключом сервера, без подмены сертификатов и без прокси. Она натолкнула автора нынешней статьи на мысль — можно ли применить такой подход к Java приложениям, использовав вместо файла сессионных ключей отладочные записи JVM? Оказалось — можно, и я расскажу, как это сделать.

# Summary. An optional shortened abstract.
summary: Кулинарные заметки о вскрытии перехваченного SSL-трафика в Wireshark

tags:
- криптография
- wireshark
- на-русском
- ssl
- логи
featured: false

links:
- name: "Хабр"
  url: "https://habr.com/ru/post/254205/"
url_pdf: ''
url_code: 'https://github.com/Toparvion/nss-java-maker'
url_dataset: ''
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
image:
  caption: 'Авторский арт'
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
{{% callout info %}}
Эта статья имеет расширение и продолжение в виде [статьи в журнале Хакер](/publication/2015/xakep/).
{{% /callout %}}
