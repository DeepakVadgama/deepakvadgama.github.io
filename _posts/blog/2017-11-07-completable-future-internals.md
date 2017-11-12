---
layout: post
title: Java CompletableFuture internals
category: blog
comments: true
published: false
excerpt: Grokking over Java async programming tool
tags: 
  - development
  - java
---

## Introduction

In Java, it has always been easy to execute a task in different thread using Runnable/Callable. It helps in offloading task from main thread (eg: In Android, network requests are not allowed be executed in a UI thread). 

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
 
What happens if we want to execute a task once its dependant task is completed? 

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

This is exactly the use-case ```CompletableFuture``` was built for.
It can be used to chain multiple dependant tasks. It is very similar to JavaScript Promises which helps chain the call-back methods (aka tasks). 

{% highlight java %}
 CompletableFuture.supplyAsync(() -> getStockInfo(“GOOGL”))   // executed in a thread-pool 
        .whenComplete((info, exec) -> System.out.println(info))  
        .thenApplyAsync(Stock::getRate)
        .thenAccept(rate -> System.out.println(rate))
        .thenRun(() -> System.out.println(“done”)));
{% endhighlight %}

Another way to think of it is Java Streams where each step can be executed in async manner (in a separate thread). 

You can explore more about CompletableFuture in [this talk](https://www.youtube.com/watch?v=Q_0_1mKTlnY).

Now that we understand its use case, let's walk-through its [source code](https://github.com/openjdk-mirror/jdk/blob/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent/CompletableFuture.java).

## Core logic 

### Chained calls

The USP of this class is chained calls (aka fluid API). For this to work, each call should return 
instance of a ```CompletableFuture```. 

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

### Wrapping the task

Every task supplied is wrapped in an internal class. This class is responsible for 

- executing the supplied task
- setting the result value or exception or null (in case its a Run method)
- calling the postComplete method so that next CompletableFuture can be called

{% highlight java %}

// not actual class, just representation. Actual classes detailed below.
static final class AsyncTask extends ForkJoinTask{
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
        
        // call the post complete call-back to trigger the next task in the chain
        d.postComplete();
    }
}
{% endhighlight %}

### Storing the chain



## Compute methods

### supplyAsync

### whenComplete

### thenApply

### theRun

# Conclusion

Massive API

Hit me up in the comments for any queries or corrections.
