---
layout: post
title: Java HashMap code walk-through
category: blog
comments: true
published: false
excerpt: Grokking over Java's HashMap implementations
tags: 
  - development
  - java
---

## Introduction

I love ['Concurrency in Practice'](https://g.co/kgs/WT7WVy). Its a Java concurrency book written by 
[Brian Goetz](https://www.linkedin.com/in/briangoetz),
 [Doug Lea](https://en.wikipedia.org/wiki/Doug_Lea) and 
[Joshua Bloch](https://en.wikipedia.org/wiki/Joshua_Bloch). It is 
considered a definitive guide on the subject. These fine folks were also involved in [JSR166](https://jcp.org/en/jsr/detail?id=166) 
and have authored many of the concurrency/collection classes in JDK.

It is fascinating to walk through their code. There is lot to learn about code structure, performance, trade-offs etc. 

Let's start with HashMap (and its cousins) and its evolution from JDK 5/6 to 8.

**Side note:** If you want to understand how HashMaps are implemented at basic level, I highly recommend this video
from [Go](https://www.youtube.com/watch?v=Tl7mi9QmLns) 

## Basics

Array with pointers to LinkedLists. 

Does it store the hash value in the node. 

Threshold (aka Load Factor) of 0.75 for resizing the array. 
It basically allocates new array with twice the size, repopulates the new array (Does it synchronize during this operation?)
using modulo operator. 

First hash check then equals check for all. Thus, the O(N) worst case i.e. when all the keys have same hash code and are thus part of same linked list. Though average case is O(1).

## Bins vs Trees

Java 8 
Changes beneficial if keys are Comparable
http://openjdk.java.net/jeps/180

Treeify 
Untreeify 
MinTableForTreeify

Trees possibility of loosing root node during iterator.remove(). What about lists?

Trees worst case performance of O(log n) since trees are balanced. 

Though overhead of twice the space, and if programming code is bad, then twice the time. 

## Go's use of bit map


## Probability of list of size k

Unless programming mistakes or malicious code, the hashcodes follow a [poisson distribution](https://en.wikipedia.org/wiki/Poisson_distribution)


## Learnings

- For a core data structure being used billions (probably trillions) of times, its okay to introduce complexity
to gain extra performance. 
- There are always trade offs between performance and space overhead
- 