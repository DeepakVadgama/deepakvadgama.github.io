---
layout: page
title: Photography
excerpt: "Photographs taken by Deepak Vadgama"
---

<div>
    <ul>
        <li><a href="https://photos.app.goo.gl/wcmkwsCUf5pm7LDN7">Australia</a></li>
        <li><a href="https://goo.gl/photos/zrwS7KyrwY6mGXZd8">Portfolio</a></li>
        <li><a href="https://goo.gl/photos/VvTjmMLYmCjkLkH49">Floral</a></li>
        <li><a href="https://photos.app.goo.gl/yMOMq1M29GXFXZ0s1">Gujarat</a></li>
        <li><a href="https://goo.gl/photos/n3u4pTLCPqa4CfBP6">Mangalore</a></li>
        <li><a href="https://photos.app.goo.gl/PSN1XEkZj15KopVE2">London</a></li>
        <li><a href="https://photos.app.goo.gl/1ZmRyWhmKbDxlz033">NYC</a></li>
        <li><a href="https://photos.app.goo.gl/gjUiHV2UxV1dmujB3">Singapore, Malaysia, Thailand</a></li>
    {% for post in site.categories['photography'] %}
        {% if post.url %}
            <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endif %}
    {% endfor %}
    </ul>
</div>
