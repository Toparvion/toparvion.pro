---
title: "AppCDS for Spring Boot applications: first contact"
authors:
- toparvion
date: "2019-10-14T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2019-12-18T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["0"]

# Publication name and optional abbreviated publication name.
publication: "[On Medium](https://medium.com/@toparvion/appcds-for-spring-boot-applications-first-contact-6216db6a4194)"
publication_short: ""

abstract: |
  **Application Class Data Sharing (AppCDS)** is a JVM feature for startup acceleration and memory saving. Originated from HotSpot of JDK 1.5 (back in 2004), it had been staying quite limited and partly commercial for a long time. But in OpenJDK 10 (2018), it became widely available and significantly more applicable. Moreover, recently released Java 13 made the feature application even more simple.

  The idea behind AppCDS is to “share” once loaded classes between JVM instances on the same host. It seems that it should be fit for microservices, especially for Spring Boot “broilers” that have thousands of library classes, because the JVM would not need to load (parse and verify) those classes on every start of every instance, and those classes would not duplicate in memory. Consequently, the startup should become faster and memory footprint should be smaller. Sounds great, doesn’t it?

  That’s right. But if you’re used to not believing street banners but checked facts and measures, then welcome aboard! Let’s find out what it’s like truly…

# Summary. An optional shortened abstract.
summary: A summary of the research how to use AppCDS with Spring Boot

tags:
- spring-boot
- appcds
- performance
featured: false

links:
- name: "Medium"
  icon: medium
  icon_pack: fab
  url: "https://medium.com/@toparvion/appcds-for-spring-boot-applications-first-contact-6216db6a4194"
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
This article was also [published](/en/post/2019/10/appcds-with-spring-boot/) on this site to improve its appearance. There is also [Russian translation](/post/2019/10/appcds-with-spring-boot/) of the article.
{{% /alert %}}
