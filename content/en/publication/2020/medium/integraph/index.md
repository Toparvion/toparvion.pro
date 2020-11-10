---
title: "How to visualize a Spring Integration graph with Neo4j?"
authors:
- toparvion
date: "2020-04-29T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2019-05-05T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["0"]

# Publication name and optional abbreviated publication name.
publication: "[On Medium](https://medium.com/@toparvion/how-to-visualize-a-spring-integration-graph-with-neo4j-61927ba5bb5a?source=friends_link&sk=35ce8c20b150bdaad2eb238f1cb3bbaf)"
publication_short: ""

abstract: |
  Spring Integration framework is able to [represent](https://docs.spring.io/spring-integration/docs/5.2.5.RELEASE/reference/html/system-management.html#integration-graph) the current state of all [EIP](https://www.enterpriseintegrationpatterns.com/) components and their relations in the form of a JSON graph. It seems useful for learning and debugging but unfortunately none of the googleable tools (which are just [1st](https://github.com/spring-projects/spring-flo/tree/angular-1.x/samples/spring-flo-si) and [2nd](https://ordina-jworks.github.io/architecture/2018/01/27/Visualizing-your-Spring-Integration-components-and-flows.html)) gives enough flexibility for visualization and analysis of such a graph. In this article I’ll show you how to address this problem by importing the graph into the [Neo4j](https://neo4j.com/neo4j-graph-database/) graph database where such abilities are the first class citizens.


# Summary. An optional shortened abstract.
summary: Building a bridge from enterprise Java to graph databases

tags:
- spring-integration
- how-to
- neo4j
- visualization
featured: false

links:
- name: "Medium"
  icon: medium
  icon_pack: fab
  url: "https://medium.com/@toparvion/how-to-visualize-a-spring-integration-graph-with-neo4j-61927ba5bb5a?source=friends_link&sk=35ce8c20b150bdaad2eb238f1cb3bbaf"
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
{{% callout note %}}
This article was also [published](/en/post/2020/integraph/) on this site to improve its appearance. There is also [Russian translation](/post/2020/integraph/) of the article.
{{% /callout %}}
