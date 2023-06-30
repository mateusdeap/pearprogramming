---
layout: page
title: Welcome to the Pear Programming blog!
---

Here I'll leave articles on my explorations in programming (and maybe other things). Here are my latest posts:

<ul>
  {% for post in collections.posts.resources %}
    <li class="post-link">
      <a href="{{ post.relative_url }}">{{ post.data.title }}</a>
    </li>
  {% endfor %}
</ul>
