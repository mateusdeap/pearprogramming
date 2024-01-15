---
layout: page
title: Posts
---

<div class="list">
{% for post in collections.posts.resources %}
    <a href="{{ post.relative_url }}">{{ post.data.title }}</a>
{% endfor %}
</div>

