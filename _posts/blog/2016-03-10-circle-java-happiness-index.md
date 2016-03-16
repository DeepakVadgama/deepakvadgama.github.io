---
layout: post
title: Circle of development - Abstractions or Simplicity  
category: blog
comments: true
published: false
excerpt: Java ecosystem, language abstractions and terseness. 
---

<figure>
    <a href="{{ site.url }}/images/blog/monolith.png"><img src="{{ site.url }}/images/blog/monolith.png"></a>
</figure>

I worked as a Java developer on a trading system for close to 2 years. 

It was a fairly large project. The application connected to 5 different markets each with different API (XML, FIX, proprietary etc). To massage the trade, application had to access close 7 systems, outside our purview, to access reference data. The code base itself was around few hundred thousand lines of code. 

We faced lot of issues due to the monolithic nature of our architecture
