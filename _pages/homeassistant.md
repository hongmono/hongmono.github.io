---
permalink: /homeassistant/
title: "Homeassistant"
layout: archive
---

{% assign posts = site.categories.homeassistant %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %}{% endfor %}