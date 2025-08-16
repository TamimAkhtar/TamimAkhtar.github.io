---
title: Projects
icon: fas fa-code-branch
order: 6
layout: page
---

<ul>
  {% assign projects = site.posts | where_exp: "post", "post.categories contains 'projects'" %}
  {% for post in projects %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span class="post-date">({{ post.date | date: "%Y-%m-%d" }})</span>
    </li>
  {% endfor %}
</ul>