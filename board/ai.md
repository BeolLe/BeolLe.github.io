---
title: "AI 게시판"
layout: page          # 테마에 맞게 'page' / 'mydoc_page' 등으로 바꿔도 됨
sidebar: home_sidebar # 형님이 실제 쓰는 사이드바 이름
permalink: /boards/ai/
---

# AI 게시판

AI 관련 글 목록입니다.

{% for post in site.categories.ai %}
- [{{ post.title }}]({{ post.url }})  
  {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
