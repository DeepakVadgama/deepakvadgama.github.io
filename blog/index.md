---
layout: page
title: Blog
excerpt: "A List of blog posts by Deepak Vadgama"
---

<div>
    <ul>
    {% for post in site.categories['blog'] %}
        {% if post.url %}
            <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endif %}
    {% endfor %}
    </ul>
</div>