---
layout: default
title: Home
---

# Welcome to My Blog

안녕하세요! Yejin의 블로그에 오신 것을 환영합니다.

## Recent Posts

<ul class="post-list">
{% for post in site.posts limit:5 %}
  <li class="post-list-item">
    <h2>
      <a class="post-link" href="{{ post.url | relative_url }}">
        {{ post.title }}
      </a>
    </h2>
    <p class="post-meta">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
    <p class="post-excerpt">{{ post.excerpt }}</p>
  </li>
{% endfor %}
</ul>
