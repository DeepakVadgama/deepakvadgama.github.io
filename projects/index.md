---
layout: page
title: Projects
excerpt: "Projects created by Deepak Vadgama"
---

<div>
    <ul>
    {% for post in site.categories['projects'] %}
        {% if post.url %}
            <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endif %}
    {% endfor %}
    </ul>
</div>
