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


With ```ExecutorService```, it is even easier to manage multiple uniform tasks in separate a thread/thread-pool.

{% highlight java %}
// 10 threads executing 100 tasks.
ExecutorService executorService = Executors.newFixedThreadPool(10);
for(int i=0; i<100; i++){
    executorService.execute(() -> {
        System.out.println("Executing task");
    });
}
{% endhighlight %}
 
But what happens if we want to execute a dependant task once the first one is completed. 

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
    String companyName = future.get();  // blocking
    executorService.execute(new StockPriceTask(companyNames.get(i));
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

Now that we understand its use case, let's walk-through its [source code](https://github.com/openjdk-mirror/jdk/blob/jdk8u/jdk8u/master/src/share/classes/java/util/concurrent/CompletableFuture.java).


# Conclusion

Massive API

Hit me up in the comments for any queries or corrections.
