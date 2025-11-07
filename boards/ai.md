---
title: "AI 게시판"
layout: page       
sidebar: home_sidebar
permalink: /boards/ai/g
---

# AI 게시판

AI 관련 글 목록입니다.

{% for post in site.categories.ai %}
- [{{ post.title }}]({{ post.url }})  
  {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
