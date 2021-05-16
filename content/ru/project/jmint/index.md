---
title: jMint
summary: Java-агент для внедрения тестовых поведений в приложения
tags:
- тестирование
- отладка
- байткод
date: "2016-08-01T00:00:00Z"
weight: 20

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: ""
  focal_point: Smart

links:
  - icon: github
    icon_pack: fab
    name: Проект на GitHub
    url: https://toparvion.github.io/jmint/
---

**jMint** – это инструмент для разработки и тестирования, реализующий подход [Side Effect Injection](/event/2018/jbreak/) – внедрение сторонних поведений в приложение в runtime с целью его отладки или тестирования. jMint представляет собой Java-агент, умеющий модифицировать байт код целевого приложения таким образом, чтобы включать в его методы (до, после или вместо них) различные вкрапления отладочного/тестового кода. Эти вкрапления называются **дроплетами** и представляют собой исходный код на Java, написанный с учетом будущего внедрения в целевой код. Подробнее о написании дроплетов см. в [документации](https://toparvion.github.io/jmint/) к проекту.
