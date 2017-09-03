---
layout: post
title: Java logging facades and bridges
category: blog
comments: true
published: true
excerpt: Making sense of seemingly complicated java logging dependencies
tags: 
  - java
  - logging
---

Logging is one of the most fundamental aspects of any language.
Unfortunately, Java's in-built logging mechanism ```java.util.logging``` (available since JDK 1.4) was [not feature rich nor flexible enough](https://stackoverflow.com/a/11360517/3494368) for most use-cases.
The community created multiple libraries to fill in this gap.

After more than a decade, the logging ecosystem has become quite complicated with each library using its own logging mechanism. Even standard projects' dependencies look like this.

{% highlight java %}
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:1.5.2.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:1.5.2.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:1.5.2.RELEASE:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.1.11:compile
[INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.1.11:compile
[INFO] |  |  |  +- org.slf4j:jcl-over-slf4j:jar:1.7.24:compile
[INFO] |  |  |  +- org.slf4j:jul-to-slf4j:jar:1.7.24:compile
[INFO] |  |  |  \- org.slf4j:log4j-over-slf4j:jar:1.7.24:compile
{% endhighlight %}

and some conversations make no sense.

<figure style="max-width: 600px; margin-left: auto; margin-right: auto">
    <a href="{{ site.url }}/images/blog/logging/spring-logging.png"><img src="{{ site.url }}/images/blog/logging/spring-logging.png"></a>
</figure>


Let us try to understand how we got here.

If you are new to Java logging, refer to [this article](https://www.loggly.com/ultimate-guide/java-logging-basics/)
to understand its features.

## Basics

```log4j``` was most used logging library in the beginning.
Once configured, using it in the code was relatively simple.

{% highlight java %}
import org.apache.log4j.Logger;

public class LogExample {

    final static Logger logger = Logger.getLogger(LogExample.class);

    private void run(){
        logger.info("important log statement");
    }
}
{% endhighlight %}

## Facades

Though, using log4j classes directly in the code causes tight coupling.
The solution was to introduce [Simple Logging Facade for Java](https://en.wikipedia.org/wiki/SLF4J) aka ```slf4j```;
a library which delegated the calls to the underlying logging implementation (eg: log4j or logback) based on runtime binding.

<figure>
    <a href="{{ site.url }}/images/blog/logging/slf4j.png"><img src="{{ site.url }}/images/blog/logging/slf4j.png"></a>
</figure>

The code changes to this.
{% highlight java %}
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogExample {

    private static Logger logger = LoggerFactory.getLogger(LogExample.class);

    private void run(){
        logger.info("important log statement");
    }
}
{% endhighlight %}


## Bridges

Unfortunately the community could not standardize on single facade to use.
Many Java libraries used different facades (eg: ```JCL``` aka Jakarta Commons Logging) or in many cases direct implementations (ignoring the tight coupling problem).

This created a very hairy issue.
Suppose you want to create a project and log with combination of ```slf4j``` and ```logback```.
So far so good.
But what if you want to import a library say, Guava, and that in turn uses ```java.util.logging```
How would your ```slf4j``` or ```logback``` know about that library's logging?

<figure>
    <a href="{{ site.url }}/images/blog/logging/logging_bridge_problem.png"><img src="{{ site.url }}/images/blog/logging/logging_bridge_problem.png"></a>
</figure>

What we need is a ```bridge``` (aka adaptor) library, if you will, which would redirect all those logs called via ```java.util.logging``` to ```slf4j```

<figure>
    <a href="{{ site.url }}/images/blog/logging/logging_bridge_solution.png"><img src="{{ site.url }}/images/blog/logging/logging_bridge_solution.png"></a>
</figure>

Problem solved. Now as long as you have the right bridge(s) for all the logging libraries
being used (directly and indirectly) in your project. All the logs will be flown/streamed
into ```slf4j``` and into your final logging implementation.

```slf4j``` has all the bridges you will ever need.

<figure>
    <a href="{{ site.url }}/images/blog/logging/slf4j_bridges.png"><img src="{{ site.url }}/images/blog/logging/slf4j_bridges.png"></a>
</figure>
*[Image Source](https://www.slf4j.org/images/legacy.png)*

The best part of this solution is -- all the bridges are bound/configured automatically at runtime.
Ofcourse, the problem of which permutation-combination to use will still require including/excluding the right bridges/facades from your pom.xml. Though framework developers do [think hard](https://jira.spring.io/browse/SPR-14512) to make it easy for the developers.

Hit me up in the comments for any queries or critiques.