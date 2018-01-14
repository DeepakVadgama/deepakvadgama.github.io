---
layout: post
title: Java DelayQueue internals
category: blog
comments: true
published: true
excerpt: Grokking over DelayQueue source code
tags: 
  - java
  - source-code-walkthrough
---

## Introduction

[DelayQueue](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/DelayQueue.html) is used as a work-queue for storing scheduled tasks in [ScheduledExecutorService](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ScheduledExecutorService.html). 

Its core logic is simple: The elements are stored in a priority queue, sorted based on their scheduled time (earliest to expire is at head of the queue). During poll/take operations, the head element can be returned only if/when it's scheduled time has expired. 

Though, this can potentially mean multiple threads synchronizing on the head element causing overhead. 
This is resolved using ```Leader-Follower``` pattern which is improvement on ```Half-Sync/Half-Async``` pattern. 
This makes the ```DelayedQueue``` implementation quite fascinating. 

The source code for ```DelayQueue``` can be found [here](http://gee.cs.oswego.edu/cgi-bin/viewcvs.cgi/jsr166/src/main/java/util/concurrent/DelayQueue.java?view=markup). Leader-Follower paper can be found [here](http://www.cs.wustl.edu/%7Eschmidt/PDF/lf.pdf)

## Leader-Follower pattern

Suppose there is an scheduled task at the head of the queue with timeout of 5 seconds. All the threads wanting 
to take/poll the element have to sleep (aka timed-wait) for 5 seconds. Once 5 seconds are over and element 
is eligible to be taken out of the queue, all threads will vie for same element. 

Instead, one thread is chosen as leader at the beginning, and only that threads awaits 5 seconds.
All other threads will wait on a ```condition```. Once 5 seconds are over, the leader will take the element, 
and signal the ```condition``` so that someone else can become the leader and get the next element in the queue.  

<figure>
    <a  href="{{ site.url }}/images/blog/leader_follower_pattern.png" data-lightbox="image-1"><img src="{{ site.url }}/images/blog/leader_follower_pattern.png"></a>
</figure>

Steps:

1. Leader awaits for the head element to expire
2. All other threads await on a condition
3. Leader takes the element after expiry & signals the condition
4. One of the other threads becomes the new leader

The above algorithm should become clearer as we walk-through the source code below.

## Delayed interface

```DelayedQueue``` stores instances of ```Delayed``` instances. 

{% highlight java %}
public interface Delayed extends Comparable<Delayed> {

     // return the remaining delay
     // if value < 0, the delay has already elapsed
    long getDelay(TimeUnit unit);
}

// sample implementation (partial code)
public class MyDelayedTask implements Delayed {

    public long getDelay(TimeUnit unit) {
        return unit.convert(time - System.nanoTime(), NANOSECONDS);
    }
    
    public int compareTo(Delayed other) {
    
        // check remaining time
        long diff = time - x.time;
        if (diff < 0)
            return -1;
        else if (diff > 0)
            return 1;
            
        // in case both have same expiry
        // check for which arrived earlier (FIFO)
        else if (sequenceNumber < x.sequenceNumber)
            return -1;
        else
            return 1;
    }
}
{% endhighlight %} 

## Initialization 

{% highlight java %}

 // lock for the synchronization
 private final transient ReentrantLock lock = new ReentrantLock();
  
 // queue to store all delayed elements (sorted by expiry/delay time remaining)
 private final PriorityQueue<E> q = new PriorityQueue<E>();

 // reference to leader thread
 private Thread leader = null;
  
 // condition for other threads to wait on
 private final Condition available = lock.newCondition();
 
{% endhighlight %} 

## Add / Offer

{% highlight java %}
    public boolean add(E e) {
        return offer(e);
    }

    public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
        
            // add element to the queue
            q.offer(e);
            
            // if this new element is new head of the queue 
            // i.e. having shorter expiry time
            // reset leader and signal condition 
            if (q.peek() == e) {
                leader = null;
                available.signal();
            }
            return true;
        } finally {
            lock.unlock();
        }
    }
{% endhighlight %} 

## Poll

Get the element at head of the queue if expired else return null.

{% highlight java %}

    public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
        
            // get head of the queue
            E first = q.peek();
            
            // if there are no elements or if none have expired yet return null 
            if (first == null || first.getDelay(NANOSECONDS) > 0)
                return null;
            else
                return q.poll();
        } finally {
            lock.unlock();
        }
    }
    
{% endhighlight %} 

## Take

Get the element at head of the queue if expired else block until expiry.

{% highlight java %}

public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
    
        for (;;) {
        
            // 1. get head element 
            E first = q.peek();
            
            // 2. if there is no element, wait for the signal
            if (first == null)
                available.await();
            else {
            
                // 3. if first element has expired return the element
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return q.poll();
                    
                first = null; // don't retain ref while waiting
                
                // 4. if leader is already chosen, wait for the signal
                if (leader != null)
                    available.await();
                else {
                
                    // 5. set this thread as the leader 
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    
                    // 6. wait for the element to expire
                    try {
                        available.awaitNanos(delay);
                    } finally {
                    
                        // 7. once expired, reset the leader
                        // run through for loop, and return the element
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
    
        // 8. if leader is reset and there are still elements in the queue
        // signal the other waiting threads
        if (leader == null && q.peek() != null)
            available.signal();
            
        lock.unlock();
    }
}
{% endhighlight %}

## Poll

Get element at head of the queue if expired else wait for given duration or element's expiry whichever is shorter. 

{% highlight java %}
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
        
            // 1. get head element 
            E first = q.peek();
            
            if (first == null) {
                // 2. if there is no element, and timeout has occurred, return null
                if (nanos <= 0)
                    return null;
                
                // 3. if there is no element, wait on condition, 
                // but, only until timeout or signal whichever is first
                else
                    nanos = available.awaitNanos(nanos);
            } else {
            
                // 4. get expiry of the head element
                long delay = first.getDelay(NANOSECONDS);
                
                // 5. if expired return the element
                if (delay <= 0)
                    return q.poll();
                    
                // 6. if timeout has occurred return null
                if (nanos <= 0)
                    return null;
                    
                first = null; // don't retain ref while waiting
                
                // 7. if given timeout is less than head element expiry
                // then wait for condition but only for maximum of given timeout
                if (nanos < delay || leader != null)
                    nanos = available.awaitNanos(nanos);
                else {
                
                    // 8. if given timeout is more than element's expiry & there is no leader
                    // set current thread as the leader 
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                    
                        // 9. wait until first element expires
                        long timeLeft = available.awaitNanos(delay);
                        
                        // 10. update the timeout accordingly
                        nanos -= delay - timeLeft;
                        
                    } finally {
                    
                        // 11. once expired, reset the leader
                        // run through for loop, and repeat
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
    
        // 12. if leader is reset and there are still elements in the queue
        // signal the other waiting threads
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
{% endhighlight %}


## Conclusion

We skipped few methods, but even then this is one of the shorter source codes we've seen so far. I was initially 
lost as to why we needed a leader and a condition. Going through the paper helped. 

Hit me in the up in the comments for any queries or concerns. 