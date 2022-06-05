---
layout: default
title: Home
description: "Home page"
permalink: /
nav_exclude: true
---

# Recent activity
{% assign sorted = site.pages | sort:'date' | reverse %}
{% for page in sorted %}
{% if page.grand_parent == 'Capture the Flag' or page.parent == 'Hack The Box' or page.parent == 'blog' %}  
{{ page.date | date: "%d %b %Y" }}: {{ page.title }} - {{ page.description }}  <a href="{{ page.url }}">LINK</a>  |
{% endif %}
{% endfor %}
