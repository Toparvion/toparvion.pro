---
title: "Когда тормозит Java? Владимир Плизга. Алексей Рагозин. Павел Кислов"
event: "Spring АйО Подкаст №59"
event_url: https://t.me/spring_aio_chat/29893

location: Подкаст (онлайн)
# address:
  # street: ул. Пирогова 1
  # city: Санкт-Петербург
  # region: CA
  # postcode: '630090'
  # country: Россия

summary: Эпизод подкаста об анализе производительности приложений на JVM
abstract: |   
    00:00:00 — Интро, представление гостей  
    00:00:40 — Что такое профайлер  
    00:01:43 — Зачем нужен профайлер: реальный кейс с лагами  
    00:06:38 — Насколько профайлеры вообще нужны разработчикам  
    00:08:53 — Опыт без профайлеров и первый жесткий фейл  
    00:12:13 — Какие бывают методы профилирования  
    00:13:04 — С чего начать анализ производительности  
    00:16:02 — Сэмплирующий профайлер: как он работает  
    00:17:18 — Flame Graph и поиск узких мест  
    00:18:08 — Когда сэмплирования недостаточно  
    00:19:34 — Трассировка и как она работает  
    00:21:25 — Профилирование памяти и heap dump  
    00:22:05 — Ошибки новичков при профилировании  
    00:23:32 — Экзотические виды профилирования  
    00:24:01 — Обзор инструментов: VisualVM, Async Profiler, JFR  
    00:27:14 — Open-source vs коммерческие профайлеры  
    00:29:15 — Как вообще научиться профилировать  
    00:30:52 — Практика, обучение и реальные кейсы  
    00:33:50 — Истории учеников и ускорение в 20 раз  
    00:35:18 — Курсы по профилированию и обучение  
    00:37:10 — Spring АйО Академия и обучение  
    00:38:50 — Профилирование vs observability (трейсинг)  
    00:40:07 — Разница между профайлингом и tracing  
    00:41:27 — Когда что использовать  
    00:42:40 — Continuous profiling и современные подходы  
    00:47:18 — «Профайлер — как мотоцикл Ява»  
    00:47:48 — Профилирование вне JVM (Python, C++)  
    00:49:23 — Универсальные подходы к профилированию  
    00:51:37 — Что нового в мире профайлинга  
    00:52:27 — Новые возможности Java Flight Recorder  
    00:55:28 — Инструментирование в Java 25  
    00:57:29 — Улучшения sampling в JFR  
    00:59:15 — Обновления Async Profiler  
    01:02:04 — Почему профилирование все еще сложно  
    01:02:18 — Искусственный интеллект и профилирование  
    01:03:54 — AI-инструменты для анализа performance  
    01:05:55 — Генерация SQL для heap dump через LLM  
    01:08:54 — Где AI реально помогает  
    01:10:43 — Проблемы LLM в профилировании  
    01:12:07 — MCP, агенты и будущее инструментов  
    01:13:25 — Какие AI использовать (ChatGPT vs DeepSeek)  
    01:15:36 — Шутки про использование AI  
    01:16:25 — Будущее  
    01:16:41 — Завершение  

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2026-04-13T12:00:00Z"
date_end: "2026-04-13T13:30:00Z"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2026-04-21T19:00:00Z"

authors:
  - toparvion
tags:
  - java
  - performance
  - profiler
  - mat
  - visualvm
  - dump
  - jvm

# Is this a featured talk? (true/false)
featured: false

image:
  caption: 'Spring АйО Подкаст №59'
  focal_point: Smart

links:
 - icon: spotify
   icon_pack: fab
   name: Spotify
   url: https://open.spotify.com/episode/7i0eldXIuigwyPUW81M3Iy?si=0y4wDGKtQ7WGTlIOeTsTkQ
 - icon: yandex
   icon_pack: fab
   name: Яндекс.Музыка
   url: https://music.yandex.ru/album/35180719
 - icon: apple
   icon_pack: fab
   name: Apple Podcasts
   url: https://podcasts.apple.com/ru/podcast/spring-%D0%B0%D0%B9%D0%BE/id1812906540?i=1000747833310
 - icon: vk
   icon_pack: fab
   name: ВКонтакте
   url: https://vkvideo.ru/video-226426202_456239263?list=ln-jtXKR8N5g14rLWV49A
 - icon: youtube
   icon_pack: fab
   name: YouTube
   url: https://youtu.be/tVyf-mI1LTE

# url_code: "https://github.com/Toparvion/fat-jar-sample"
# url_pdf: ""
# url_slides: "https://speakerdeck.com/toparvion/spring-boot-fat-jar-tonkiie-chasti-tolstogho-artiefakta"
# url_video: "https://youtu.be/ib_yvGhuzDg?t=5095"

# Markdown Slides (optional).
#   Associate this talk with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = []
  

# Enable math on this page?
math: false
---
