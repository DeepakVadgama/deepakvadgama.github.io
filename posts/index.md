---
layout: page
title: All Posts
excerpt: "A List of Posts"
---

<br/>
{% for category in site.categories %}
  <div>
  <h3>{{ category | first | capitalize }}</h3>
    <ul>
    {% for posts in category %}
      {% for post in posts %}
	{% if post.url %}		
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
	{% endif %}
      {% endfor %}
    {% endfor %}
    </ul>
  </div>
{% endfor %}
