---
layout: page
title: Welcome!
---

Here I'll leave articles on my explorations in programming (and maybe other things). Here are my latest posts:

<div class="list">
{% for post in collections.posts.resources %}
    <a href="{{ post.relative_url }}">{{ post.data.title }}</a>
{% endfor %}
</div>

