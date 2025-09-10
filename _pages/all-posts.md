---
layout: single
title: "All Posts"
permalink: "/all-posts/"
---

{% assign posts = site.posts %}
{% if posts.size == 0 %}
_No posts in this topic yet. Check back soon._
{% else %}
<ul>
{% for post in posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    — <small>{{ post.date | date: "%b %-d, %Y" }}</small>
  </li>
{% endfor %}
</ul>
{% endif %}
