---
title: 🐞 BugClinic
summary: Лабораторное веб-приложение для поиска багов производительности
tags:
- тестирование
- производительность
date: "2024-10-01T00:00:00Z"
weight: 15

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: ""
  focal_point: Smart

links:
  - icon: github
    icon_pack: fab
    name: Проект на GitHub
    url: https://github.com/Toparvion/spring-petclinic-rest
---

BugClinic &mdash; это форк демонстрационного приложения [Spring PetClinic REST](https://github.com/spring-petclinic/spring-petclinic-rest), содержащий сознательно внедрённые (и подробно описанные) баги производительности. Форк используется как основа практической части в проводимых мною [тренингах](/training/performance) по анализу проблем производительности приложений на JVM.

### Описания практических кейсов

Все кейсы (баги производительности) описаны по одной схеме:

1. Легенда (краткое описание бизнес-логики кейса)
2. Техническая реализация (краткое описание имплементации)
3. Проблема
4. Шаги воспроизведения
5. Объяснение (разгадка проблемы с комментариями)
6. Варианты решения (чаще всего несколько)

Для удобства воспроизведения и анализа информация по каждому кейсу представлена в двух PDF-файлах:

* `description.pdf` содержит первые три пункта из списка выше;
* `solution.pdf` – последние три пункта.

Все классы практических кейсов объявлены в [пакете](https://github.com/Toparvion/spring-petclinic-rest/tree/master/src/main/java/org/springframework/samples/petclinic/service/perf) `org.springframework.samples.petclinic.service.perf` и попадают в Spring-контекст лишь при наличии соответствующих аргументов запуска (см. описания).

#### По потокам

* Кейс №1
  * [Описание](threads/case-1/description.pdf)
  * [Решение](threads/case-1/solution.pdf)
* Кейс №2
  * [Описание](threads/case-2/description.pdf)
  * [Решение](threads/case-2/solution.pdf)
* Кейс №3
  * [Описание](threads/case-3/description.pdf)
  * [Решение](threads/case-3/solution.pdf)
* Кейс №4
  * [Описание](threads/case-4/description.pdf)
  * [Решение](threads/case-4/solution.pdf)

{{% callout note %}}

Описания кейсов по другим темам (память, JFR) ещё не оформлены для публикации. Если хотите увидеть их поскорее, [сообщите](/#contact) мне, пожалуйста, об этом.

{{% /callout %}}