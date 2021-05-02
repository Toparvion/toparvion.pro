---
title: NSS Java Maker
summary: CLI-утилита для вскрытия TLS-трафика с помощью логов JVM
tags:
- криптография
date: "2015-08-14T00:00:00Z"
weight: 50

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: ""
  focal_point: Smart

links:
- icon: github
  icon_pack: fab
  name: Проект на GitHub
  url: https://github.com/Toparvion/nss-java-maker
# url_code: "https://github.com/Toparvion/nss-java-maker"
---

**NSS Java Maker** &mdash; небольшая консольная утилита, автоматизирующая часть операций по дешифрации дампа TLS (SSL) траффика, перехваченного сетевым сниффером [Wireshark](https://www.wireshark.org/).

Более подробно о процессе в целом можно почитать в следующих  авторских статьях:

* [Дешифрация TLS трафика Java приложений с помощью логов](https://habr.com/ru/post/254205/) (Хабр)
* [Лог всемогущий. Расшифровываем TLS-трафик с помощью JVM](https://xakep.ru/2015/08/14/log-almighty/) (журнал Хакер)

{{% callout warning %}}
Разработка и тестирование утилиты велись в 2015-ом году, поэтому она может иметь проблемы совместимости с последними версиями Wireshark и/или Java.

{{% /callout %}}
