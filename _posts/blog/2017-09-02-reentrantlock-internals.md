---
layout: post
title: Java Re-entrant Lock internals
category: blog
comments: true
published: false 
excerpt: Grokking over ReentrantLock & AbstractQueueSynchronizer
tags: 
  - development
  - java
---

## Introduction

While reading source code of ArrayBlockingQueue implementation, I found this rather intriguing snippet of code.
 
{% highlight java %}
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);  // Intriguing
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
{% endhighlight %} 

 What!? That's it? Either ```true``` or ```false``` for a lock decides fairness of all blocking queue operations?
 Let's find out what makes a ReentrantLock so special. It's source code can be found [here](https://github.com/openjdk-mirror/jdk/tree/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent/locks).

## ReentrantLock & AQS

It turns out ReentrantLock creates 2 helper classes extending ```AbstractQueuedSynchronizer```, and delegates all locking operations including the fairness.

{% highlight java %}
public ReentrantLock() {
    // default is unfair sync
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}

abstract static class Sync extends AbstractQueuedSynchronizer { ... }
static final class FairSync extends Sync { ... }
static final class NonfairSync extends Sync { ... }

// delegate lock
public void lock() {
    sync.lock();
}

// delegate release
public void unlock() {
    sync.release(1);
}
{% endhighlight %}

Also, the class ```AbstractQueueSynchronizer``` aka ```AQS``` seems to be used by most locks in Java.

<figure>
    <a href="{{ site.url }}/images/blog/AQS_uses.png"><img src="{{ site.url }}/images/blog/AQS_uses.png"></a>
</figure>

Let us try to understand the code. This code walk through is not sequential nor atomic (per method). We will try to pick and choose code which will get us a working version of executors. In each subsequent step, we will add one feature or address a problem.

## Basic unfair locking

Locking means exclusive access. Integer variable called ```state``` is used to maintain this access. Using integer variable is better because if allows us to use single-instruction compare-and-swap operations/methods.

{% highlight java %}
private volatile int state;   // 0 = unlocked, >0 = locked

// Unfair Sync partial code
final void lock() {
    // if state = 0, just set the current thread as owner i.e. acquire the lock
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

// Fair Sync partial code
final void lock() {
    acquire(1);
}
{% endhighlight %}

##

{% highlight java %}
 * <pre>
 * Acquire:
 *     while (!tryAcquire(arg)) {
 *        <em>enqueue thread if it is not already queued</em>;
 *        <em>possibly block current thread</em>;
 *     }
 *
 * Release:
 *     if (tryRelease(arg))
 *        <em>unblock the first queued thread</em>;
 * </pre>
{% endhighlight %}

### Fairness
### Queueing the threads
### Barging
