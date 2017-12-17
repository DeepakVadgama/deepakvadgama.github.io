---
layout: post
title: Slow moving culture wall
category: projects
comments: true
published: true 
excerpt: Working in a large enterprise as a Lead Developer
---

Since March 2017, I have been working with one of the largest financial institutions in the world, as an Associate VP.
Contrary to my prior stints working with large enterprises, this one has been surprisingly delightful so far. 
Its mostly thanks to a small, independent & wonderful team, and a very supportive manager. 

### The Product

We own the 'Search' component of a public facing website 

- 4 developers
- ~100k unique visitors a month
- 1.3 million searches a month
- 4 tomcats across 2 data centers in hot-hot config
- Apache as reverse proxy (security, load-balancing, URL mapping etc.)
- Oracle/Cassandra (used as config store)

In the early part of the product's life, it was mandated to have entire code flow dynamic. All changes were driven 
through configuration. It was a monolith, which meant deploying any change required down-time.  By the time I joined, the monolithic system was being decoupled into components. As of today, we share our Tomcat with only one other component. 

I was able to push further towards simplicity, higher tooling and use of out-of-the-box components. 
This was also largely aided by management push to go agile/cloud-native (Thank you Amazon for scaring all the banks into it).

### Spring Boot Migration

When I joined, the Java code was pure servlet based. Using web.xml for URL mapping, reading parameters from headers manually, 
 a single (3000 line) class to manage dependencies (read: too much boilerplate). It wasn't ideal. 
I was already a big fan of [Spring Boot](http://www.deepakvadgama.com/blog/spring-boot-wonders/).
I requested to migrate entire code-base (~110k LoC) to Spring Boot. 

The migration took ~1.5 months (including test cases, integration testing, merging etc.). The migrated code-base was ~90k LoC. The code was much smaller, more manageable and more readable. I also undertook internal session on advantages of using Spring Boot. 

### Culture 

AS developers we get too comfortable using existing tools. This is especially true in the enterprise. 
I pushed for using libraries like [Spring Data](http://projects.spring.io/spring-data/), 
[AssertJ](http://joel-costigliola.github.io/assertj/) (excellent assertion library), [Mock Web Server](https://github.com/square/okhttp/tree/master/mockwebserver), [Retrofit](http://square.github.io/retrofit/) (http client), [Guava](https://github.com/google/guava) (for using time-based cache) and such. 
I am really grateful my team is far from stubborn. They all understood that these tools/libraries can hugely benefit us.  

We are still away from the ideal setup though. 
We are using Java 7 (so no lambdas), and create a WAR to deploy on Tomcat (which makes using Spring Boot Dev Tools difficult). 

There is also culture of 'what if in the future' which affects a lot of code we write. Especially pushing for things to be dynamic and config driven. 
This unnecessarily complicates the code, and in 9 months I am with this team, we have **never** updated a single config in production. With microservices and continuous deploys, I don't see the need for this. 

I am also a big fan of [Intellij IDEA](https://www.jetbrains.com/idea/). I have learned most of its shortcuts and hardly use mouse with it, which also generated interest in my team to switch from Eclipse. 

### Sessions

Ofcourse changing this culture requires little persuasion. I hosted multiple internal sessions to help everyone understand the advantages of using these tools/libraries. 
 
- [Spring Boot](http://www.deepakvadgama.com/blog/spring-boot-wonders/)
- [Git](https://drive.google.com/open?id=17wuWZCs_2DIUkMrORLXIJ615t5Tht08oYim4ztM1NGc)
- [Testing in Java](https://github.com/DeepakVadgama/java-testing)
- [Basics of Hadoop](https://drive.google.com/open?id=199HgvWRtoZLoqzsuOL-hG8sm_r6vvXWB2rrl-9HdbIQ)
- [Introduction to Cloud Computing](https://drive.google.com/open?id=1FKo5EJF4XIYATJVYA6foU4gYJobgF0ICQQjVKEMGKns)
- Jules Pipeline (Continuous Integration & Deployments)

### Symphony chatbot

I developed a 'Search' chatbot based on [Symphony platform](https://symphony.com/). 
Unfortunately, most of the details of the platform API were abstracted by our internal proprietary layer.
For each chat-message aka query, the bot retrieved results from REST API (which I built separately) and displayed results.

Having a single thread for all requests was a perfect use-case for using [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html).
In addition to offloading the task off the thread, it made the code much more readable. 

There are very limited number of users searching on the platform. We plan to use NLP in the future (for understanding user queries) if the platform usage picks up. 

### Faster development

Using Spring Profiles and offloading the startup caches on a Scheduled Executor, I was able to bring down the server startup time
**from 5 minutes to 20 seconds**. 4 developers and multiple restarts per day meant huge time savings.  
Hopefully in the future we will use [Spring Boot Dev Tools](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html) and avoid this restart altogether. 

### Conclusion

It is very important to have a good team with sufficient independence and management support (the world is on fire due to DevOps and 'Silicon Vally is coming' scare helps too). 

On a personal note, its deeply rewarding to mentor a team, witness improvements, feel appreciated and deliver solutions at a healthy clip. 

On the other hand, one day I wish to work in a great team where I'm the least nerdy/quirky/knowledgeable one. With the right culture, that's the best environment to learn, which itself is highly rewarding. I also miss working on solutions which can have much larger and positive impact. 
