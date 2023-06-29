---
layout: page
title: Posts
---

<ul>
  {% for post in collections.posts.resources %}
    <li>
      <a href="{{ post.relative_url }}">{{ post.data.title }} - {{ post.data.date }}</a>
    </li>
  {% endfor %}
</ul>
