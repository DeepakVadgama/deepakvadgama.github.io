---
layout: post
title: Oracle's Java and the poor developer
category: blog
comments: true
excerpt: 
tags: development, java
---

There is an ongoing upheaval in Java community about the lack of progress/clarity by Oracle over JEE. 
There has always been a friction between community and Oracle, but the fight is finally out in the open. 
Read [this excellent article](http://arstechnica.com/information-technology/2016/07/how-oracles-business-as-usual-is-threatening-to-kill-java/) detailing the timeline of the missteps.  

Oracle has a distinct personality. Most in tech industry have an agreed upon opinion of Oracle. 
These opinions are formed and reinforced by [court case claiming $8 billion for using 37 package APIs](http://arstechnica.com/tech-policy/2016/03/oracle-will-seek-a-staggering-9-3-billion-in-2nd-trial-against-google/), 
[Java leaders quitting Oracle](http://www.eweek.com/c/a/Application-Development/Java-Creator-James-Gosling-Why-I-Quit-Oracle-813517), 
[thrown out Java champions](https://www.infoq.com/news/2015/09/oracle-purges-java-evangelists),
[stifling alternate JVM implementations](http://arstechnica.com/information-technology/2010/10/ibm-joins-openjdk-as-oracle-shuns-apache-harmony/).. the list goes on and on. 

 <figure>
     <a href="http://www.bonkersworld.net/organizational-charts/"><img src="{{ site.url }}/images/blog/oracle_chart.png"></a>
 </figure> 
Image Source - http://www.bonkersworld.net/organizational-charts/

As the article mentions, OpenJDK and its stewards might finally break the ties with Java completely and go solo in advancing the Java platform. 
Though Java's licensing model, enterprises' insistence of using certified Java, and the effort/timescale for such undertaking might not allow such split to be fruitful.     

Side note - This feels similar to W3C vs WhatWG - where the browser vendors were tired of the slow pace of innovation in Web and formed their own committee, 
threatening the existence of W3C. W3C had to bow under pressure and agreed to form joint committee with browser vendors to address the concern. 

It would be unfair to pick on Oracle though. Even before Oracle acquired Sun, the official Java platform has been way too slow in addressing developer's needs.

## Java SE release train

Java 5 (2004) was he first major release in terms of APIs/language-updates. 
Java 6 came along couple of years later without adding much to the language.
The next major release after whopping 7 years as Java 7. Java 8 was released in 2014 while Java 9 itself has been mired in [delays](https://dzone.com/articles/oracle-announces-jigsaw-delays-push-java-9-launch) and is expected to come out in 2017.

 <figure>
     <a href="{{ site.url }}/images/blog/java_timeline.png"><img src="{{ site.url }}/images/blog/java_timeline.png"></a>
 </figure> 

## Java Logging

Its not just the Java core platform. Most of the JCPs were created in wake of open source alternatives getting popular. 
The primary and perhaps most shocking (for me) example has been Java logging.   

Logging is one of the most fundamental functionality of a language. Never baked into the language, logging frameworks were introduced by the community. 
[Log4J](http://logging.apache.org/log4j) is an amazing library for logging which has been serving Java developers well over the years. 
The community though, due to lack of consensus, created multiple competing wrappers to allow using other frameworks like [Logback](http://logback.qos.ch). 
The apparent mess is depicted below. While the official Java [solution](http://docs.oracle.com/javase/7/docs/technotes/guides/logging/) came years later with Java 7.

 <figure>
     <a href="{{ site.url }}/images/blog/logging.png"><img src="{{ site.url }}/images/blog/logging.png"></a>
 </figure> 

Try setting up log4j or logback with Tomcat, Spring MVC and varied bunch of 3rd party libraries. It can be a really painful experience. 

 
## Java EE & Java Community

Logging is just one of the many examples of this pattern: 

> community creates library -> it becomes popular -> corresponding JCP/JEP is created years later -> library is tweaked to be JCP compatible 

Here are some of more examples: 

| Community | Year  | Year | Official         |
|-----------|-------|------|------------------|
| Spring DI | 2003  | 2009 | Java DI          |
| Hibernate | 2001  | 2006 | JPA              |
| EH cache  | 2003  | 2014 | JCache           |
| Joda      | 2002? | 2014 | Java 8 date/time |

I am sure there are many more (web services, messaging, OSGI, security etc). 

I think the popularity of Java has been fueled by tremendous ecosystem/community and not the core platform itself.
    
## What next

Java is rapidly losing developer mindshare. It is chugging along thanks to large enterprises (especially in Finance industry).  

I have been looking at Job postings ([1](http://stackoverflow.com/jobs?allowsremote=true), [2](https://remoteok.io/), [3](https://weworkremotely.com/)) and hardly any startups quote Java.
Probably its too slow and complex for startups, too limiting for intellectuals (Haskell/Scala), old timers (C/C++), clarity seekers (Python), uptime freaks (Erlang), functional zealots (JavaScript/Lisp/Clojure)..  

These languages along with recent popular ones like Go, Rust, Swift, D, Rails, NodeJS have slowly started taking over.
The final nail in the coffin will be the cloud (microservices/serverless/big-query). Java will cease to be prima facie of programming languages, which might be a good thing considering the circumstances. 
