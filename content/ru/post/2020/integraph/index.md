---
title: "Как визуализировать граф Spring Integration в Neo4j?"
subtitle: ""
summary: ""
authors: 
  - toparvion
tags:
  - spring-integration
  - neo4j
  - spring-boot
categories:
  - graph-storages
date: 2020-04-14T08:31:05+07:00
lastmod: 2020-04-14T08:31:05+07:00
featured: false
draft: true

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Фреймворк Srping Integration [умеет](https://docs.spring.io/spring-integration/docs/current/reference/html/system-management.html#integration-graph-controller) отдавать текущее состояние всех [EIP](https://www.enterpriseintegrationpatterns.com/)-компонентов и их связей в виде JSON-графа. Это кажется очень удобным для анализа и отладки, но увы, ни один из нагугливаемых инструментов (а их всего-то: [раз](https://github.com/spring-projects/spring-flo/tree/angular-1.x/samples/spring-flo-si) и [два](https://ordina-jworks.github.io/architecture/2018/01/27/Visualizing-your-Spring-Integration-components-and-flows.html)) не даёт достаточно гибких возможностей для визуализации такого графа и работы с ним. В этой заметке я расскажу, как проложить мостик из мира EIP (на примере Spring Integration) в мир графовых баз данных (на примере СУБД [Neo4j](https://neo4j.com/)), где эти возможности стоят на первом месте.