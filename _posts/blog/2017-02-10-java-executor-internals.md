---
layout: post
title: Java ThreadPoolExecutor internals
category: blog
comments: true
published: true 
excerpt: Grokking over Java 8 ThreadPool Executor implementation
tags: 
  - java
  - source-code-walkthrough
---

## Introduction

Following the [fun exercise]({{site.url}}/blog/java-hashmap-internals/) to understand JDK's HashMap implementation code, I decided to walk-through the code 
for [ThreadPool executor classes](https://github.com/openjdk-mirror/jdk/tree/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent). 

This code walk through is not sequential nor atomic (per method). 
We will try to pick and choose code which will get us a working version of executors. 
In each subsequent step, we will add one feature or address a problem.
Thus more complicated aspects like locking are discussed in the latter half. 

## Table of contents
- [Core](#core)
  * [Adding tasks and new threads](#adding-tasks-and-new-threads)
  * [Worker threads creation](#worker-threads-creation)
  * [Worker Threads executing tasks](#worker-threads-executing-tasks)
  * [Reduce or Maintain pool size post-task-completion](#reduce-or-maintain-pool-size-post-task-completion)
  * [Reduce or Maintain pool size pre-task-acceptance](#reduce-or-maintain-pool-size-pre-task-acceptance)
  * [Rejection handlers](#rejection-handlers)
- [Locking](#locking)
  * [Using a common main lock](#using-a-common-main-lock)
  * [Using ctl lock](#using-ctl-lock)
  * [Worker locks](#worker-locks)
- [Executor shutdown (with states)](#executor-shutdown--with-states-)
- [Thread Interrupts](#thread-interrupts)
- [Default thread factory (Executors)](#default-thread-factory--executors-)
- [Types of executors](#types-of-executors)
- [Conclusion](#conclusion)

## Core 

Instead of starting from constructors, lets start from the heart of Executors i.e. running a submitted task.
The basic idea is, tasks submitted are added to a queue, and threads keep picking tasks from that queue and execute them. 
Based on the configuration and current number of threads, this process is slightly tweaked. 


### Adding tasks and new threads

The tasks are assigned to the threads in 3 ways

- If thread pool count < core pool size, then create new worker thread and assign task to it. 
- If thread pool count >= core pool size, add task to the queue (will be retrieved by worker thread later)
- If task queue is bounded and full, then add create new worker thread and assign task to it. 

The second argument to addWorker method just indicates the pool-size (true = corePoolSize, and false = maxPoolSize). 
  So if number of threads are more than that size, new worker thread is not added, and method returns false. 

{% highlight java %}
// partial code for execute method
public void execute(Runnable command) {

    // if no task submitted, return NPE
    if (command == null)
        throw new NullPointerException();
        
    // if number of threads in the pool is less than core-pool-size, 
    // then create a new worker thread and assign the new task to it
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
    }
    
    // else, add task to the queue
    if (workQueue.offer(command)) {
        ....
    }
    
    // else create new thread and assign task to it
    else if (!addWorker(command, false)) // pool-size is >= core-pool-size
        reject(command);   // If pool-size >= max-pool-size
}
{% endhighlight %}

### Worker threads creation

In books and tutorials related to Java Threads, we are shown a new Thread instance is generally created by passing argument of a Runnable. In this class, the Runnable (Worker) creates its own thread from the thread-factory (see constructor) and holds a reference to the same.   

Once the worker is created, it is added to set of workers, representing the thread-pool. The worker is then started 
and its status is returned to the caller. 

{% highlight java %}

// partial code for addWorker method
private boolean addWorker(Runnable firstTask, boolean core) {
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
    
        // 1. create a new worker with given task
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            workers.add(w);  // removed surround lock code for brevity
            t.start();
            workerStarted = true;
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    
    // 3. Return started status
    return workerStarted;
}

// partial code for Worker class
private final class Worker implements Runnable {
        
    // Thread this worker is running in. Null if factory fails.
    final Thread thread;
    
    // Initial task to run. Possibly null.
    Runnable firstTask;
    
    Worker(Runnable firstTask) {
        // the command (task) passed in section above
        this.firstTask = firstTask;
        
        // create thread from factory
        this.thread = getThreadFactory().newThread(this); 
    }

    // Delegates main run loop to outer runWorker 
    public void run() {
        runWorker(this);
    }
}
{% endhighlight %}

### Worker Threads executing tasks

As seen in code earlier, the run method of a Worker calls method runWorker. 
 This method keeps taking tasks from the queue and calls run() for each task explicitly. 
 
 Note: Process of getting tasks from queue involves more checks which we will see later. 

{% highlight java %}

// partial code for runWorker method
final void runWorker(Worker w) {

    // 1. get first task if any
    Runnable task = w.firstTask;
    w.firstTask = null;
    
    // 2. run firstTask if present or keep getting tasks from the queue, and run them
    while (task != null || (task = getTask()) != null) { 
        task.run(); // 3. call run method explicitly
        
    }
}

// partial code for getTask 
private Runnable getTask() { 

    // get new task to execute from queue, block if unavailable. 
    Runnable r = workQueue.take();
}
{% endhighlight %}

### Reduce or Maintain pool size post-task-completion

When the worker runs out of tasks to execute (queue is empty), then it calls processWorkerExit, 
which can reduce the size of the pool or can create new worker thread.
 New worker thread is created based on few conditions
 
 - If core threads are allowed to timeout, and there are no pending tasks, worker is not replaced. 
 - If current count of workers is greater than minimum required (corePoolSize), worker is not replaced.
 - If both conditions above fail, then new worker is created. 

{% highlight java %}

// partial code for runWorker method
final void runWorker(Worker w) {
    
    completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) { 
            task.run(); 
        }
        completedAbruptly = false;
    } finally {
        // if there are no more tasks to run, or if there is exception, exit.
        processWorkerExit(w, completedAbruptly);
    }
}

// partial code for processWorkerExit
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    
    // 1. If abrupt, then workerCount wasn't adjusted
    if (completedAbruptly) 
        decrementWorkerCount();

    // 2. maintain completed task count
    completedTaskCount += w.completedTasks; 
    workers.remove(w);

    if (!completedAbruptly) {
    
        // 3. Check if worker thread needs to be replaced 
        int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
        if (min == 0 && ! workQueue.isEmpty())
            min = 1;
            
        // 4. replacement not needed
        if (workerCountOf(c) >= min)
            return; 
    }
    
    // 5. replace worker 
    addWorker(null, false);
}

{% endhighlight %}

### Reduce or Maintain pool size pre-task-acceptance

While getting new tasks to execute, we may need to stop the worker based on few more conditions.

 - If shutdown is requested for the executor
 - If there are no tasks available and there is atleast 1 thread to execute tasks submitted later
 - If there are more worker threads than maximum allowed

The polling for new task, is either blocking or timeout-based depending on the thread-count (if its more than core pool size) or allowCoreThreadTimeOut (if core threads are allowed to timeout and thus reduce in number). 

{% highlight java %}
// partial code for getTask 
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // return null if thread count more than maximum, or thread timed-out polling task
        // and if there are no tasks to execute and more than one thread available
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // timeout poll if coreThreadTimeOut is allowed or thread count > corePoolSize
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
{% endhighlight %}

### Rejection handlers
 
The tasks which are rejected (due to queue full, or poolSize being full) are handled using rejectionHandlers
 
 - Caller runs Policy - calls the run method with current context (i.e. thread of the caller itself)
 - Abort Policy (default) - throw RejectedExecutionExcpetion
 - Discard Policy - do nothing (swallows the fact that it was unable to run the task)
 - Discard Oldest Policy - removes first task from queue (oldest), and enqueues the new task
 
{% highlight java %}
// partial code for execute
public void execute(Runnable command) {

    // reject if executor is not running anymore (shutdown requested)
    if (! isRunning(recheck) && remove(command))
        reject(command);
        
    // reject if task queue is full and cannot add new worker 
    if (!addWorker(command, false))
        reject(command);
}

final void reject(Runnable command) {
    handler.rejectedExecution(command, this);
}

// caller runs policy
public static class CallerRunsPolicy implements RejectedExecutionHandler {

    // run task (runnable) in the caller thread
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}

// abort policy - default
public static class AbortPolicy implements RejectedExecutionHandler {
    
    // throw rejected exception
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}

// discard policy
public static class DiscardPolicy implements RejectedExecutionHandler {

    // do nothing
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}

// discard oldest policy
public static class DiscardOldestPolicy implements RejectedExecutionHandler {

    // get first task from the queue (which will be the oldest task available)
    // then submit this particular task
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
{% endhighlight %}


## Locking

### Using a common main lock

Most of the methods in this class use a ReentrantLock called mainLock to perform synchronized access 
to the common state. 

{% highlight java %}
final ReentrantLock mainLock = this.mainLock;
mainLock.lock();
try {
    // ... code ... 
} finally {
    mainLock.unlock();
}
{% endhighlight %}

### Using ctl lock

This class uses an AtomicInteger to maintain combined state of 2 fields 

- Number of worker threads (29 bits)
- Run state of the executor (2 bits)

Updating of the worker thread count and the state of the executor is then performed using compareAndSet operations.

{% highlight java %}
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }

// Example code of updating state and thread count
ctl.compareAndSet(c, ctlOf(TIDYING, 0);
{% endhighlight %}

### Worker locks

Worker class extends AbstractQueuedSynchronizer to gain locking mechanism. 
Many of the JDK Concurrent utilities like Semaphore, ReEntrantLock, CountdownLatch extend the aforementioned class. 
Worker class needs to override only few methods, to get the desired locking (for its state).

{% highlight java %}
private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable {
  
     protected boolean isHeldExclusively() {
         return getState() != 0;
     }
    
     protected boolean tryAcquire(int unused) {
         if (compareAndSetState(0, 1)) {
             setExclusiveOwnerThread(Thread.currentThread());
             return true;
         }
         return false;
     }
    
     protected boolean tryRelease(int unused) {
         setExclusiveOwnerThread(null);
         setState(0);
         return true;
     }
    
     public void lock()        { acquire(1); }
     public boolean tryLock()  { return tryAcquire(1); }
     public void unlock()      { release(1); }
}    
{% endhighlight %}

## Executor shutdown (with states)

When a shutdown is requested for the executor, 

- it changes its state to SHUTDOWN/STOP to stop accepting new tasks
- changes it state to TIDYING 
- tries to shut all running threads (on best-effort-basis)
- changes its state to TERMINATED
- if shutdownNow is called, returns the list of pending tasks

{% highlight java %}
// partial code for shutDown
public void shutdown() {
    checkShutdownAccess();
    advanceRunState(SHUTDOWN); // change state to SHUTDOWN
    interruptIdleWorkers();
    onShutdown(); // hook for ScheduledThreadPoolExecutor
    tryTerminate();
}

// partial code for shutDownNow
public void shutdownNow() {
    
    // ... code same as shutdown...
     advanceRunState(STOP); // change state to STOP

    // return pending tasks from the queue 
    List<Runnable> tasks = drainQueue();
    return tasks;
}

// partial code for tryTerminate
final void tryTerminate() {
    interruptIdleWorkers();
    terminated();
    termination.signalAll();
}
{% endhighlight %}

## Thread Interrupts 

When the executor is shutdown, it will change its state and ask all threads to interrupt. 
Note that worker threads run the tasks using its lock, thus executor cannot interrupt such threads. 
It can only interrupt threads which are idle. 

The worker threads before starting to execute the tasks, check the state of the executor, 
and interrupt the thread. 

Thus stopping the executor is on a best effort basis. Any threads which are running long 
running tasks might take a while to respond to interrupt (or to complete the task).

{% highlight java %}
private void interruptIdleWorkers(boolean onlyOne) {
    for (Worker w : workers) {
        Thread t = w.thread;
        
        // if worker not already interrupted, try to acquire its lock
        if (!t.isInterrupted() && w.tryLock()) {
            try {
                // interrupt the thread
                t.interrupt();
            } catch (SecurityException ignore) {
            } finally {
                w.unlock();
            }
        }
    }
}

// partial code of runWorker
final void runWorker(Worker w){
 if ((runStateAtLeast(ctl.get(), STOP)))
    wt.interrupt();
}
{% endhighlight %}

## Default thread factory (Executors)

The default thread factory, creates non-daemon threads, with same priority as calling thread (or Normal priority if not set)
and sets appropriate names for the threads to distinguish between threads of different pools.  

{% highlight java %}
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}

{% endhighlight %}

## Types of executors

Now that we understand how task queue, keepAlive time, thread factory, corePoolSize and maxPoolSize are used by the executor,  creating various types of Executors is easy. 

| Type          | Meaning                                        | Min Threads | Max Threads       | Queue type          | keepAlive  |
|---------------|------------------------------------------------|-------------|-------------------|---------------------|------------|
| Fixed         | Fixed size of worker threads                   | x           | x                 | LinkedBlockingQueue | 0          |
| Single        | Single worker thread                           | 1           | 1                 | LinkedBlockingQueue | 0          |
| Cached        | Unlimited max threads                          | x           | Integer.MAX_VALUE | SynchronousQueue    | 60 seconds |
| Scheduled     | For scheduled tasks                            | x           | Integer.MAX_VALUE | DelayedWorkQueue    | 0          |

There is also an WorkStealingPool which uses [ForkJoinPool](https://github.com/openjdk-mirror/jdk/blob/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent/ForkJoinPool.java), that is a separate class not covered in this post. 

## Conclusion

~2000 lines of code for ThreadPoolExecutor class looks overwhelming at first. 
Though, if we start with only the essentials and 
keep adding each layer, it all starts to make sense. Dare I say, the code looks quite straight-forward. 
There is a sense of elegance and beauty in its simplicity. 

We have not covered [FutureTask](https://github.com/openjdk-mirror/jdk/blob/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent/FutureTask.java) and [ForkJoinPool](https://github.com/openjdk-mirror/jdk/blob/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent/ForkJoinPool.java). Those demand their own blog post. 

Hit me up in the comments for any queries or corrections.
