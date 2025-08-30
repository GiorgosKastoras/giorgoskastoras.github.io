---
layout: page
title: Projects
permalink: /projects/
---

<h3 class="posts-item-note" aria-label="Project List">Project List</h3>

{%- for project in site.projects -%}
<article class="post-item">
  <span class="post-item-date">{{ project.date | date: "%b %d, %Y" }}</span>
  <h4 class="post-item-title">
    <a href="{{ project.url }}">{{ project.title | escape }}</a>
  </h4>
</article>
{%- endfor -%}
