---
layout: post
title: Java
category: interview
comments: true
excerpt: Java interview notes
published: false
tags: 
  - interview
  - java
  - notes
---

### What do you not like about Java

- It is very difficult to write succinct code in Java. Too much boilerplate. Slightly subsidised by lambdas. 
- Threads could be less expensive akin to Go
- Message passing and Immutability are not baked-in.
- Thread communication by sharing memory is brittle (that is why so many locks are introduced), difficult to developers at all skill level to write correct code.
- I wish primitives could be directly added to collections and used as list
- Null (Kotlin fix … i have spent so much time fixing NPE and wrote so much code to avoid it). Especially in finance world where POJOs have deeply nested structure. Doing a.getB().getC().getD() cannot be written. Every object needs to be null checked. 
- No tuples (especially helpful in multiple return types which currently have to be put in HashMap)
- 
- In fact, just check the kotlin language list
- All pojos have to write getter, setter, equals, hash, clone/copy… kotlin fixes this
- No built-in light weight UI framework - They failed with JavaFX, Applets, etc. 

### What do you like about Java

- Performance (Eg: Memory allocation). Thanks to years of iterative work on the JVM it is very performant. JIT Compilation.  
- Auto GC
- Ecosystem (Vast number of mature, stable libraries and frameworks out there)
- Applicable to multiple use cases - Low Latency, Web server, Backend, etc. 
- 

### Why BigDecimal

