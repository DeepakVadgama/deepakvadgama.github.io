---
layout: post
title: Exploring concurrency in non-java languages
category: blog
comments: true
published: true
excerpt: co-routines, async-await, channels and suspending functions 
tags: 
  - java
---


<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">While I was playing with primitive Java threadpool, the world seems to have moved on to such wonderful constructs. <a href="https://t.co/YT5u8P0j5l">https://t.co/YT5u8P0j5l</a></p>&mdash; Deepak (@deepakvadgama) <a href="https://twitter.com/deepakvadgama/status/1066574109964931073?ref_src=twsrc%5Etfw">November 25, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<br/>
Recently I have been exploring concurrency features of other languages with fascination. It started with go co-routines, and then I discovered articles by [Nathaniel Smith](https://twitter.com/vorpalsmith) - creator of Python Trio library for concurrency - which piqued my interest to 11. 

Concurrency using message passing (as opposed to memory sharing) is an utterly beautiful paradigm. It will soon become relevant to Java developers with [Project Loom aka Java Fibers](http://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html) aka light-weight threads in Java.  
  
I recommend these related articles which I found most helpful

### Co-routines


- **[go concurrency patterns](https://www.youtube.com/watch?v=f6kdp27TYZs)** - This was my first introduction to co-routines. Pure elegance. 

- **[CSP paper](https://spinroot.com/courses/summer/Papers/hoare_1978.pdf)** - This paper was the foundation of co-routines in go.    

- **[Timeouts and cancellation for humans](https://vorpus.org/blog/timeouts-and-cancellation-for-humans/)** - How timeouts should be implemented in IO libaries and extended to use cancellation tokens.  

- **[Notes on Structured Concurrency](https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/)** - Once you grasp concept of goroutines, this article is a must read. It addresses pitfalls of goroutines. This was also a reference for Kotlin's [coroutine implementation](https://kotlinlang.org/docs/reference/coroutines-overview.html). I have never read such clear explanation of such difficult topic before. 

- **[Python Trio concurrency library](https://www.youtube.com/watch?v=i-R704I8ySE)** - Introduction to Python's Trio library (which uses structured concurrency) by its creator Nathaniel Smith. 

- **[What color is your function?](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)** - After promises/futures and rx libraries, async-await is the newest tool being added in many languages. This article explores how some languages force developers to label asynchronous functions all they way up the call stack. 

## Bucket list

In addition to coroutines, there are some features I plan to explore soon


- **Borrow Checker (Rust)** - How Rust enforces variable ownership/lifetimes at compile time, in effect making language safer at runtime.

- **yeid/generator functions (Dart/Kotlin)** - Functions that generate asynchronous streams of values.

- **Dispatchers (Kotlin)** - Choosing the pool/thread where the async function runs/returns. More useful for Android/JS with single UI thread I think. 

### Kotlin 
I have high hopes for Kotlin. It incorporates many of the above mentioned [concurrency features](https://kotlinlang.org/docs/reference/coroutines/coroutines-guide.html). Also, being source compatible with Java, it acts as an [easier ramp](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/coroutines-guide.md) onto these wonderful constructs. 

If you have any similar recommendations please let me know in the comments. 

