---
layout: page
title: 모든 태그 모아보기
permalink: /tags/
search: true
sidebar: home_sidebar
---

<div class="tag-cloud-container" style="padding: 20px; background: #f8f9fa; border-radius: 8px; border: 1px solid #ddd;">
    <h3 style="margin-top: 0;"><i class="fa fa-tags"></i> 태그 클라우드</h3>
    <div class="tag-cloud">
        {% assign tags = site.tags | sort %}
        {% for tag in tags %}
        <a href="#{{ tag[0] | slugify }}" class="tag-item" style="font-size: {{ tag[1] | size | times: 2 | plus: 100 }}%; margin: 5px; display: inline-block; text-decoration: none;">
            <span class="label label-default">{{ tag[0] }} ({{ tag[1] | size }})</span>
        </a>
        {% endfor %}
    </div>
</div>

<hr>

<div class="tag-list">
    {% for tag in tags %}
    <div id="{{ tag[0] | slugify }}" style="margin-top: 30px;">
        <h4><i class="fa fa-hashtag"></i> {{ tag[0] }}</h4>
        <ul style="list-style-type: none; padding-left: 10px;">
            {% for post in tag[1] %}
            <li style="margin-bottom: 5px;">
                <span class="label label-info" style="margin-right: 10px; font-size: 0.8em;">{{ post.date | date: "%Y-%m-%d" }}</span>
                <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
            </li>
            {% endfor %}
        </ul>
    </div>
    {% endfor %}
</div>