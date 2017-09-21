---
layout: post
title: Java Reentrant Lock internals
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

## Table of contents

- [ReentrantLock & AQS](#reentrantlock---aqs)
- [Locking](#locking)
  * [Basic locking](#basic-locking)
  * [Algorithm for concurrent access](#algorithm-for-concurrent-access)
  * [Fair acquire](#fair-acquire)
  * [Add to the FIFO queue](#add-to-the-fifo-queue)
  * [Node Exclusivity](#node-exclusivity)
  * [Barging](#barging)
  * [Unfair acquire](#unfair-acquire)
- [Unlocking](#unlock)
  * [Basic unlocking](#basic-unlocking)
  * [When multiple threads are holding lock](#when-multiple-threads-are-holding-lock)
  * [What about queued threads](#what-about-queued-threads)
- [Conclusion](#conclusion)

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

Let us try to understand the code. This code walk through is not sequential nor atomic (per method). In each subsequent step, we will add one feature or address a problem.

## Locking

### Basic locking

Locking means exclusive access. Integer variable called ```state``` is used to maintain this access. Using integer variable is better because if allows us to use single-instruction compare-and-swap operations/methods.

{% highlight java %}
private volatile int state;   // 0 = unlocked, >0 = locked

// basic lock acquire
final void lock() {
    // if state = 0, just set the current thread as owner i.e. acquire the lock
    if (compareAndSetState(0, 1)){
        setExclusiveOwnerThread(Thread.currentThread());
}
{% endhighlight %}

### Algorithm for concurrent access

For locking operation, if lock is already being used, the basic algorithm is to enqueue the thread and block.
Similarly for unlock operation, release the lock (reset the state) and unblock first queued thread.

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

### Fair acquire

{% highlight java %}
// try to acquire the lock, if unable to do so, enqueue the thread.
public final void acquire(int arg) {
    // partial code
    if (!tryAcquire(arg)) {
        addWaiter(Node.EXCLUSIVE), arg);
    }
}

// fair Sync to try and acquire
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();

    if (c == 0) {
        // if state = 0 (unlocked), and if no threads are queued (waiting to acquire a lock)
        // then reset the state and acquire the lock
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // if current threads itself is the owner of the lock,
    // then update the state value (in case of ReentrantLock, add 1) and continue
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // else return false so that this thread can be then added to the wait queue
    return false;
}
{% endhighlight %}

### Add to the FIFO queue

If the ```tryAcquire``` fails, then add the thread to the queue and block it until
the lock is released and it is at head of the queue (all earlier threads are removed from the queue).

{% highlight java %}
private Node addWaiter(Node mode) {
    // create a new node with current thread
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // full enqueue
    enq(node);
    return node;
}
{% endhighlight %}

### Node Exclusivity

You may have noticed ```Node.EXCLUSIVE``` in the code snippet above.
This is easy to understand if we understand its sibling ```Node.SHARED```.
Shared node is used for read locks where multiple threads can simultaneously have access to the lock.
In most cases we use ```Node.EXCLUSIVE```, especially in our context of ```ReentrantLock``` where-in we need exclusive access.

### Barging

Now that we understand how queues are used, lets look at what is ```Barging```.

Lets revisit the snippet of code.

{% highlight java %}
// unfair Sync partial code
final void lock() {
    // don't care about the potential queued threads
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
}
{% endhighlight %}

Suppose the state was just unlocked, and there are few threads waiting in the queue.
But, suddenly this new thread tries to acquire the lock, and it does not check the queue state, and
gets to acquire the lock, unfairly. This is also called ```Barging```.

### Unfair acquire

Unfair acquire is almost same, except, it again tries to barge in without checking the queue state.

{% highlight java %}
// unfair Sync partial code
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // don't bother with state of the queue
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // rest is same as fair tryAndAcquire
}
{% endhighlight %}

## Unlock

### Basic unlocking

Now that the thread owns a lock, unlocking/releasing it is simpler.

{% highlight java %}
public void unlock() {
    sync.release(1);
}

// partial modified code for both fair/unfair sync
protected final boolean tryRelease(int releases) {
    setState(0); // reset the state
    setExclusiveOwnerThread(null); // remove the ownership
    return true;
}
{% endhighlight %}

### When multiple threads are holding lock

The code changes slightly when multiple threads are holding lock (eg: Read lock).
In this case, only 1 of the thread might release the lock, thus state cannot be set to 0,
and ownership is still retained by the thread.

{% highlight java %}
// partial code for both fair/unfair sync
protected final boolean tryRelease(int releases) {

    // update the state (in case of ReentrantLock, subtract 1)
    int c = getState() - releases;
    boolean free = false;
    // if state = 0 (unlocked), only then remove ownership.
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    // update the state
    setState(c);
    return free;
}
{% endhighlight %}

### What about queued threads

If there are threads waiting to acquire the lock, we need to ```unpark``` the thread at head of the queue (waited the longest).

{% highlight java %}
public final boolean release(int arg) {
    // if lock was successfully released, remove a waiting thread from queue
    if (tryRelease(arg)) {
        Node h = head;
        // if queue is not empty (i.e. head is not null) and
        // wait status = 0 (default value, other values are for Condition and such)
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

// partial and updated code for unpark
private void unparkSuccessor(Node node) {
    if (node != null)
        LockSupport.unpark(node.thread);
}
{% endhighlight %}

## Conclusion

I was putting off going through this code for a long time. It turned out to be a wonderful ride.
We didn't walk-through some parts of the code like ```Condition``` object, doAcquireInterruptibly, Cancelled status and lot more.

Hats off to the skills of the JDK developers (especially likes of Doug Lea and Brian Goetz).
The code is easy (relatively speaking) to understand and the documentation for these classes is the most comprehensive and informative I've ever encountered.

Hit me up in the comments for any queries or corrections.
