---
layout: default
title: Blog
---

## Blog

Welcome to my blog! Here I'll share updates on projects, thoughts on machine learning, computational neuroscience, and other topics I find interesting.

<ul>
  {% for post in site.posts %}
    <li>
      <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <p><small>Published on {{ post.date | date: "%B %d, %Y" }}</small></p>
      {{ post.excerpt }}
      <p><a href="{{ post.url | relative_url }}">Read more...</a></p>
    </li>
  {% endfor %}
</ul>

{% if site.posts.size == 0 %}
  <p>Failed to load blog.</p>
{% endif %}
