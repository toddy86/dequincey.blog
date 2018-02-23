---
layout: photos
title: "Photos"
permalink: /galleries
---

Photos from my world travels, organised by location. All of my photos were taken with my phone (currently the Huawei P9)

{% for gallery in site.data.galleries %}
- [{{ gallery.name }}]({{ gallery.id }})
{% endfor %}
