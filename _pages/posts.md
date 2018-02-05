---
title: "All posts"
layout: archive
permalink: /posts/
---

{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}

{% include paginator.html %}