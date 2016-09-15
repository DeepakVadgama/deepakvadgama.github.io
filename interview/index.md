---
layout: page
title: Interview
excerpt: "Interview preparation"
---

<div>
    <ul>
    {% for post in site.categories['interview'] %}
        {% if post.url %}
            <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endif %}
    {% endfor %}
    </ul>
</div>
