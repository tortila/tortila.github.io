---
layout: archive
permalink: /elsewhere/
title: "Me elsewhere"
---

{% include group-by-array collection=site.external_publications field="category" %}

{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
  <h2>
    <a href="{{ post.external_url }}">
      {{ post.name }}
    </a>
  </h2>
  <p>{{ post.content | markdownify }}</p>
  {% endfor %}
  <br>
{% endfor %}
