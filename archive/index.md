---
layout: default
title: Blog Archive
type: archive
---

## Here are all {{ site.posts.size }} posts.

<ul>
  {% for post in site.posts %}
    <li>
      <time class="date" datetime="{{ page.date | date: "%Y-%m-%d" }}">{{ post.date | date: "%b %d, %Y"  }}</time> &mdash; <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
