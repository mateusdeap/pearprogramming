---
layout: page
title: Posts
---

<div class="list">
  <% collections.posts.resources.each do |post| %>
    <a href="<%= post.relative_url %>"><%= post.data.title %></a>
  <% end %>
</div>

