---
title: "Лог всемогущий. Расшифровываем TLS-трафик с помощью JVM"
authors:
- toparvion
date: "2015-07-22T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2015-08-14T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: "[В журнале Хакер](https://xakep.ru/2015/08/14/log-almighty/)"
publication_short: ""

abstract: |
  Совершая платеж в Интернет-магазине или ином финансовом сервисе, ты наверняка инициируешь SSL-соединение где-то на серверной стороне с участием какого-нибудь Java-приложения. А теперь представь: что, если тебе нужно исследовать это соединение? В силу бизнес-ценности его нельзя сделать открытым даже в тестовом окружении. Устроить [MITM](https://ru.wikipedia.org/wiki/%D0%A7%D0%B5%D0%BB%D0%BE%D0%B2%D0%B5%D0%BA_%D0%BF%D0%BE%D1%81%D0%B5%D1%80%D0%B5%D0%B4%D0%B8%D0%BD%D0%B5) с помощью [Fiddler](https://en.wikipedia.org/wiki/Fiddler_%28software%29)’а не даст привязка к настоящим сертификатам, и даже если ты раздобудешь приватный ключ сервера, успех не гарантирован.

  Тупик? Оказывается, нет! Трафик такого приложения можно расшифровать, если у тебя есть его перехват [Wireshark](https://ru.wikipedia.org/wiki/Wireshark)’ом и… логи JVM.

# Summary. An optional shortened abstract.
summary: Теория и практика по вскрытию траффика от JVM в Wireshark

tags:
- криптография
- wireshark
- на-русском
- ssl
- логи
featured: true

links:
- name: "Журнал Хакер"
  url: "https://xakep.ru/2015/08/14/log-almighty/"
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
  caption: 'Иллюстрация из [журнала](https://xakep.ru/wp-content/uploads/2015/08/dots.png)'
  focal_point: Smart
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects:
- nss-java-maker

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
# slides: example
---
{{% alert info %}}
Эта статья является расширением и продолжением предыдущей [статьи на Хабре](/publication/2015/habr/).
{{% /alert %}}
