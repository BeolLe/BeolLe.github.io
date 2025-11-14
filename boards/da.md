---
title: "Data Analysis 게시판"
layout: page       
permalink: /boards/da/
---

# Data Analysis 게시판

Data Analysis 관련 글 목록입니다.

{% for post in site.categories.da %}
- [{{ post.title }}]({{ post.url }})  
  {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
