---
layout: single
title: "All Posts"
permalink: "/all-posts/"
---

{% assign sorted = site.posts | sort: "date" | reverse %}
<ul>
{% for post in posts %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> — <small>{{ post.date | date: "%b %-d, %Y" }}</small></li>
{% endfor %}
</ul>
