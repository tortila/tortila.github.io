---
layout: single
permalink: /cv/
title: "My CV"
slug: cv
---

{% assign cv_files = site.static_files | where: "name", "cv.pdf" %}
{% for file in files %}
  {{ file.path }}
{% endfor %}