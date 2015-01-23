---
layout: page
title: Poems
excerpt: "A List of Poems by Deepak Vadgama"
---

<div>
    <ul>
    {% for post in site.categories['poem'] %}
        {% if post.url %}
            <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endif %}
    {% endfor %}
    </ul>
</div>