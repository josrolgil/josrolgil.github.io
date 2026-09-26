---
layout: page
title: Tags
permalink: /tags/
---

<div class="tags-list">
  {% assign sorted_tags = site.tags | sort %}
  {% for tag in sorted_tags %}
    {% assign tag_name = tag[0] %}
    <a href="#{{ tag_name | slugify }}" class="tag-chip">{{ tag_name }} ({{ tag[1].size }})</a>
  {% endfor %}
</div>

{% for tag in sorted_tags %}
  {% assign tag_name = tag[0] %}
  <h2 id="{{ tag_name | slugify }}">{{ tag_name }}</h2>
  <ul class="tag-posts">
    {% for post in tag[1] %}
      <li>
        <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
        <small>{{ post.date | date: "%b %-d, %Y" }}</small>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
