---
layout: post
title: Java CompletableFuture internals
category: blog
comments: true
published: true
excerpt: Grokking over Java async programming tool
tags: 
  - development
  - java
---

- [Introduction](#introduction)
- [Core logic](#core-logic)
  * [Chained calls](#chained-calls)
  * [Executors](#executors)
  * [Types](#types)
  * [AsyncSupply](#asyncsupply)
  * [Setting the result](#setting-the-result)
  * [Chained methods](#other-chained-methods)
  * [Running the chained methods](#running-the-chained-methods)
- [Other APIs](#other-apis)
  * [Manual Complete](#manual-complete)
  * [Cancel](#cancel)
- [Conclusion](#conclusion)

## Introduction

In Java, it has always been easy to execute a task in separate thread using Runnable/Callable. It helps in offloading task off the main thread (eg: In Android, network requests are not allowed be executed in a UI thread). 

{% highlight java %}
Runnable task = new Runnable(){ 
    public void run(){
        System.out.println("Want to run this in separate thread.");
    }
}

Thread thread = new Thread(task);
thread.start();
{% endhighlight %} 


With ```ExecutorService```, it is even easier to manage multiple independent tasks in separate a thread/thread-pool.

{% highlight java %}
// 10 threads executing 100 tasks.
ExecutorService executorService = Executors.newFixedThreadPool(10);
for(int i=0; i<100; i++){
    executorService.execute(() -> {
        System.out.println("Executing task");
    });
}
{% endhighlight %}
 
Though, if we want to execute multiple related/dependant tasks one after another, 
the flow is not completely asynchronous. It is because the future.get method (which is used to wait for a task to be completed) is a blocking operation.   

{% highlight java %}
List<Future> companies = new ArrayList<Future>();

ExecutorService executorService = Executors.newFixedThreadPool(10);
for(int i=0; i<100; i++){
    Future future = executorService.submit(new CompanyTickerTask(companyNames.get(i));
    companies.add(future);
}

for(Future future : companies){
    // blocking operations performed in order of futures stored in the list 
    // instead of futures which are completed 
    String companyTicker = future.get();  // blocking
    executorService.execute(new StockPriceTask(companyTicker);
}
// skipping boiler plate defining task classes  
{% endhighlight %}

<figure style="border: 1px solid gray">
    <a  href="{{ site.url }}/images/blog/completable_future.png" data-lightbox="image-1"><img src="{{ site.url }}/images/blog/completable_future.png"></a>
</figure>

This is exactly the use-case ```CompletableFuture``` was built for; to chain multiple dependant tasks. 
It is very similar to JavaScript Promises which helps chain call-back methods (aka tasks). 

{% highlight java %}
 CompletableFuture.supplyAsync(() -> getStockInfo(“GOOGL”))   // executed in a thread-pool 
        .thenApplyAsync(Stock::getRate)   // callback method once above lambda (getStockInfo) is completed
        .thenAccept(rate -> System.out.println(rate)) 
        .thenRun(() -> System.out.println(“done”)));
{% endhighlight %}

Another way to think of it is Java Streams where each step can be executed in async manner (in a separate thread). 
You can explore more about CompletableFuture in [this talk](https://www.youtube.com/watch?v=Q_0_1mKTlnY).

Now that we understand its use case, let's walk-through its [source code](https://github.com/openjdk-mirror/jdk/blob/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent/CompletableFuture.java).

## Core logic 


### Chained calls

The USP of this class is chained calls (aka fluid API). For this to work, each call should return 
instance of a ```CompletableFuture``` so that same methods can be again applied on the return type. 

{% highlight java %}
public static CompletableFuture asyncSupplyStage(Executor e, Supplier f) {

    // create new instance 
    CompletableFuture<Void> d = new CompletableFuture<Void>();
    
    // execute the task on an executor
    e.execute(new AsyncSupply(d, f));
    
    // return the instance to allow chained calls
    return d; 
}
{% endhighlight %}

### Executors

The compute methods have 2 versions, 

- one which takes ExecutorService instance as an argument to run the task.
- one which only supplies the task, and CompletableFuture uses its own ExecutorService to run it.
 
{% highlight java %}
// Default executor
private static final Executor asyncPool = useCommonPool ?
    ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();

// Fallback if ForkJoinPool.commonPool() cannot support parallelism
static final class ThreadPerTaskExecutor implements Executor {
    public void execute(Runnable r) { new Thread(r).start(); }
}

// Use own thread-pool 
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}

// Use externally supplied thread-pool
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
}
{% endhighlight %}

### Types

There are 4 types of primary/compute calls supported.

- **Supply**: Used to supply value. No input, provides output. 
- **Apply**: Used to apply a function on an input. Takes input, provides output.
- **Accept**: Used to accept a value and run a function using the same. Takes input, doesn't provide output.
- **Run**: Used to run a function. No input, no output. 

Each of these are represented as classes: ```AsyncSupply```, ```UniApply```, ```UniAccept``` and ```UniRun```.

### AsyncSupply

Lets start with ```CompletableFuture.supplyAsync```. It is responsible for

- executing the supplied task
- setting the result value or exception or null (in case its a Run method which doesn't have any output)
- calling the postComplete method so that next CompletableFuture can be called

{% highlight java %}

// the public method which uses default thread-pool (which is instance of ForkJoin default pool)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}

// the public method which uses supplied thread-pool
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
}

static <U> CompletableFuture<U> asyncSupplyStage(Executor e, Supplier<U> f) {
    // create a new instance to return
    CompletableFuture<U> d = new CompletableFuture<U>();
    
    // submit the applied function to the thread-pool
    e.execute(new AsyncSupply<U>(d, f));
    
    // return the instance so that chained call can be triggered
    return d;
}

// Partial code from actual class
static final class AsyncSupply extends ForkJoinTask {
    public void run(){
        try {
            // run the supplied function and get the value
            Value val = f.run();
            
            // set the CompletableFuture value 
            // to be used by next CompletableFuture in the chain as input
            d.completeValue(val);
            
        } catch (Throwable ex) {
            // set the value as exception 
            d.completeThrowable(ex);
        }
        
        // call the postcomplete which triggers the the next task (callback method) in the chain
        d.postComplete();
    }
}
{% endhighlight %}

### Setting the result

The result of a function (output) if any, is applied to a single field using CAS (compare-and-swap) operation. 
This set result is then used by subsequent CompletableFutures as input

{% highlight java %}
// Completes with a non-exceptional result, unless already completed
final boolean completeValue(T t) {
    return UNSAFE.compareAndSwapObject(this, RESULT, null, (t == null) ? NIL : t);
}

// null results are wrapped
static final AltResult NIL = new AltResult(null);

// Exceptions within the CompletableFuture are also set in same method 
final boolean completeThrowable(Throwable x) {
    return UNSAFE.compareAndSwapObject(this, RESULT, null, encodeThrowable(x));
}
static AltResult encodeThrowable(Throwable x) {
    return new AltResult((x instanceof CompletionException) ? x : new CompletionException(x));
}
{% endhighlight %}

### Other chained methods

The other chained methods (thenApply, thenAccept, thenRun) are slightly different than the ```supply``` method.
They are very similar to each other though. Lets take a look at one of them, say ```thenApplyAsync``` method.

{% highlight java %}
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn) {
    return uniApplyStage(asyncPool, fn);
}

private <V> CompletableFuture<V> uniApplyStage(Executor e, Function<? super T,? extends V> f) {
    CompletableFuture<V> d =  new CompletableFuture<V>();
    
    // check if previous CompletableFuture is already completed. 
    // if yes, just run the apply function and return the instance of this CompletableFuture
    // if no, get inside if
    if (e != null || !d.uniApply(this, f, null)) {
    
        // create instance of UniApply with source and dependant CFs and the function to apply.
        UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
        
        // push the instance on stack so that it can be popped and run later
        push(c);
        
        // again check if its prior future is completed, if so apply the function 
        c.tryFire(SYNC);
    }
    
    // return the instance to run chained methods.
    return d;
}

// partial code 
final <S> boolean uniApply(CompletableFuture<S> a,
                           Function<? super S,? extends T> f,
                           UniApply<S,T> c) {
    Object r; Throwable x;
    
    // if prior result is not completed (a.result) return false. 
    if ((r = a.result) == null)
        return false;
        
    // if result is completed
    tryComplete: if (result == null) {
    
        // if the set result is an Exception, there is no point in going on, 
        // just call the completeThrowable again and return
        if (r instanceof AltResult) {
            if ((x = ((AltResult)r).ex) != null) {
                completeThrowable(x, r);
                break tryComplete;
            }
            r = null;
        }
        
        // use the set result as input to our function f, apply the function, 
        // and call completeValue with the result (output)
        try {
            S s = (S) r;
            completeValue(f.apply(s));
        } catch (Throwable ex) {
            // if there is an exception, same drill, call completeThrowable
            completeThrowable(ex);
        }
    }
    return true;
}
{% endhighlight %}

### Running the chained methods

Notice that the CompletableFuture dependencies are only added to the stack if the previous ones are not completed yet.
In such cases, the stack has to be popped one after other, and run. 

Lets revisit the ```postComplete``` method from the AsyncSupply's run method. 

{% highlight java %}
// partial code
final void postComplete() {
    // On each step, variable f holds current dependents to pop and run. 
    CompletableFuture<?> f = this; Completion h;
    
    // until stack is empty
    while ((h = (f = this).stack) != null) {
        CompletableFuture<?> d; Completion t;
        
        // change head of the stack to next node
        if (f.casStack(h, t = h.next)) {
            // run the function for the CompletableFuture (whatever it may be)
            f = (d = h.tryFire(NESTED)) == null ? this : d;
        }
    }
}
{% endhighlight %}

## Other APIs

### Manual Complete

```CompletableFuture``` also exposes a public method to manually set the value from the outside. 
 
{% highlight java %}
public boolean complete(T value) {

    // same method seen before, called directly with the result value
    boolean triggered = completeValue(value);
    
    // call the postComplete method so that dependant CFs can be called. 
    postComplete();
    
    return triggered;
}
{% endhighlight %}
     
### Cancel

Similarly it also allows a CF instance to be cancelled manually. 

{% highlight java %}
public boolean cancel(boolean mayInterruptIfRunning) {
    
    // wrap cancel exception in a result wrapper
    AltResult theResult = new AltResult(new CancellationException());
    
    // set it only if result is not already set
    boolean cancelled = (result == null) && internalComplete(theResult);
    
    // trigger postComplete so that dependant CFs can be called
    postComplete();
    
    return cancelled || isCancelled();
}
{% endhighlight %}


## Conclusion

```CompletableFuture``` class source is ~2400 lines long. It also has massive API. 
Also, sadly, of all the JDK source classes I've read, this one was the most difficult to understand. 
Anyways, it was satisfying to finally _get_ it. 

We skipped all teh compound methods which uses multiple CompletableFuture instances in combinations of and, or, any etc. 
Though, hopefully, once you get the basic flow detailed above, the rest should come relatively easy. 

Hit me up in the comments for any queries or corrections.
