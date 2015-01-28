---
layout: page
title: Photography
excerpt: "Photographs taken by Deepak Vadgama"
---

<div>
    <ul>
    {% for post in site.categories['photography'] %}
        {% if post.url %}
            <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endif %}
    {% endfor %}
    </ul>
</div>
