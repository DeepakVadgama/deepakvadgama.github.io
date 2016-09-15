---
layout: post
title: Java wait, notify and locking
category: blog
comments: true
published: false
excerpt: Explanation of wait/notify using Producer-Consumer problem.
tags: development, java, concurrency
---

Had a fun conversation with a friend recently. It was about how Java's wait/notify affects thread locking. 

<figure>
 <a href="/images/blog/java-wait-notify.png"><img src="/images/blog/java-wait-notify.png"></a>
</figure>

## Doubts

## How it works




[8:38 PM, 7/5/2016] Prabhat Kuber: Dude!! I was just looking at the wait n notify code and got serious doubt.
Assume that we have two synchronized functions having lock on same object, once function contains only notify and another contains only wait function. We have two threads calling each function at the same time. Assume that function first( containing wait) got called and get the lock, and function second would be deprived of lock and would be waiting for the lock bt then thread one will call the wait method and lock would be released. Now, thread 2 would acquire the lock of the object and start processing den at the juncture of existing from the function it would call the notify.. till now I am clear .

Now doubt start engulfing me...
1)  The moment notify would be called, does thread 2 will release the lock, does thread 1 would try to acquire the lock. ?
2) does lock would be released only successful completion of second function?
3) No fairness policy is dere, does by any chance, some other thread would acquire the lock instead of thread 1?
4) if thread1, got the lock , then there might b the chance of inconsistency, if partially executed function have some global variable. Doesn't it break the very sacrilege purpose it has been use?
6) if there 2 acquire the lock then chance of dieing of thread one is 100%? Dieing mean waiting for the notify forever.
[11:19 PM, 7/5/2016] +91 99201 22644: 1. Notify in 2nd thread, will make first thread runnable again but 2nd thread will not release lock until it is out of its synchronized block

2. Correct

3. Hmm, I thought thread would go in first in first out queue.. but yeah, if there is not fairness policy, some other thread can acquire the lock

4. Yes, it's possible that global state will change, but that's not violating purpose. Purpose of the locking is sharing data between threads. So one thread can definitely change state while other has only partially finished. Think of producer consumer problem. If global change (availability of data)  is unchanged after consumer thread comes out of wait, then it won't work. But yes. Responsibility is of coder to change state properly.

6. if notify is called by 2 threads, both will release lock and go into waiting area.  And yes, if there is no 3rd thread which calls notify, both will remain in that state forever.
[11:23 PM, 7/5/2016] Prabhat Kuber: Thanks dude..
[11:23 PM, 7/5/2016] +91 99201 22644: Anytime buddy
[11:24 PM, 7/5/2016] Prabhat Kuber: Dude!! If 2nd thread become runnable again, where is code which tells the thread to get a lock
[11:24 PM, 7/5/2016] Prabhat Kuber: ?
[11:24 PM, 7/5/2016] Prabhat Kuber: Does our JVM internally manage it?
[11:25 PM, 7/5/2016] +91 99201 22644: That's our synchronized keyword in method signature right
[11:25 PM, 7/5/2016] +91 99201 22644: JVM will try to run it again since it's runnable but immediately see the synchronized block and wait for the object lock
[11:25 PM, 7/5/2016] Prabhat Kuber: Dude!! Thread already bypassed it and called wait method and released d lock
[11:27 PM, 7/5/2016] +91 99201 22644: Yeah, but even took continue after wait() line of code, since while method is synchronized.. JVM will remember the line to resume from but will still wait for the lock

