---
title: "Computer Science 게시판"
layout: page       
permalink: /boards/cs/
---


CS 관련 글 목록입니다.

{% for post in site.categories.cs %}
- [{{ post.title }}]({{ post.url }})  
  {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
