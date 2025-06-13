---
title: Notes
icon: fas fa-stream
order: 5
layout: page
---

<ul>
  {% assign notes = site.posts | where_exp: "post", "post.categories contains 'notes'" %}
  {% for post in notes %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span class="post-date">({{ post.date | date: "%Y-%m-%d" }})</span>
    </li>
  {% endfor %}
</ul>