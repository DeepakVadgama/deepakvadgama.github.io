---
layout: page
title: Posts by Tags
excerpt: "An archive of posts sorted by tag."
---

{% capture site_tags %}{% for tag in site.tags %}{{ tag | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
{% assign tags_list = site_tags | split:',' | uniq | sort %}
{% for tag in tags_list %}
  {% assign t = tag %}
  {% assign posts = site.posts %}

  <p style="margin: 0 0 0.225rem; font-weight: bold">{{ t | downcase | remove:',' }}</p>
  <ul style="margin-top: 0.5em;margin-bottom: 1em;">
  {% for post in posts %}
    {% if post.tags contains t %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endif %}
  {% endfor %}
  </ul>
{% endfor %}
