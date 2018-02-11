---
layout: post
title: Java ArrayDeque internals
category: blog
comments: true
published: false
excerpt: Grokking over ArrayDeque source code
tags: 
  - java
  - source-code-walkthrough
---

## Introduction

[ArrayDeque](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html) is doubled-ended-queue. That means elements can be added/removed from the top and the bottom. This means, this data structure can also be used as Stack and List. 

Documentation says it is generally faster than Stack & LinkedList (when used as a queue). Makes sense because Stack class was written in JDK 1.0 and implemented using Vector which uses synchronization. I am assuming ArrayDeque is faster than LinkedList when used as a queue, because former is backed by an array while latter with a list of nodes.

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/jroper?ref_src=twsrc%5Etfw">ArrayDeque makes a great stack, queue, or deque.</p>&mdash; Joshua Bloch (@joshbloch) <a href="https://twitter.com/joshbloch/status/584272140830056450?ref_src=twsrc%5Etfw">April 4, 2015</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

Let's walk through ArrayDeque [source code](http://gee.cs.oswego.edu/cgi-bin/viewcvs.cgi/jsr166/src/main/java/util/ArrayDeque.java?view=co). 

## Initialization

{% highlight java %}

    


    public ArrayDeque() {
        elements = new Object[16];
    }
{% endhighlight %}