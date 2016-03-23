---
layout: post
title: Refactoring to sanity
category: projects
comments: true
excerpt: Enhancing Balaji Extrusion's ERP software (based on Spring MVC).
---

One of my first undertakings as a [freelancer/consultant](http://deepakvadgama.com/blog/circle-into-tangent/) has been working on a client's existing internal ERP software (based on Java 7, Spring MVC, Hibernate, Tomcat). 
The software is used by all departments to manage end-to-end flow from Sales Orders to Invoices and deliveries, and thus is fairly important for the company. 
While it has been very interesting working on new features, I had to spend 50% of the effort on refactoring the code back to healthy state. 
Fun snippets of code I started with can be found at the end of the post. 

### Making of chaos
The client initially opted for SAP based solution which turned out to be too expensive a proposition to maintain. They lost close to INR 12 lakhs.
Later, they gave the project to a large Indian services company. The project was delivered 6 months late, with cost of INR 15 lakhs (~$22,000).
Sadly the project was run in its entirety by 3 developers fresh out of college.
Suffice to say by the time the project came to me, though it was in working state, the codebase was in ugly shape.
 
Since most of the issues could've been easily avoided with proper guidance to these freshers, 
let me jot down *instead-do-this* advice-list for issues I encountered.

### Setup of Project
- Ensure codebase is maintained in some external & private repository. [BitBucket](http://bitbucket.org) and [Gitlab](http://gitlab.com) provide it for free.
- Setup logging. Java's logging ecosystem is a mess. Use [this chart](http://www.slf4j.org/images/legacy.png) for guidance, or better yet, use out-of-the-box solutions like [Spring Boot](http://projects.spring.io/spring-boot/).
- Ensure logging is setup using appropriate rollovers and max-file-sizes while retaining atleast 2 weeks of history. [Log4j](https://logging.apache.org/log4php/docs/appenders/rolling-file.html) or [Logback](http://stackoverflow.com/a/14199642/3494368) references. 
- Regulate Maven dependencies using [this plugin](https://maven.apache.org/plugins/maven-dependency-plugin/analyze-mojo.html). Over time, projects accumulate unused dependencies which increases size of output artifact (WAR/JAR).
- For new projects, make it mandatory to have test cases, and schedule deliverables accordingly.
- Create [profiles](https://docs.spring.io/spring-boot/docs/.../boot-features-profiles.html) in code, to avoid accidentally connecting to Prod/UAT database.

### Coding
- Use enums instead of Strings when possible. They restrict values and reduce equals errors, especially case sensitive ones.
- Prefer primitives over wrapper class if possible. Otherwise stick to wrapper classes, avoid mixing them too much.
- Object conversions (eg: POJO to DTO) should be done within owner classes. If done outside, it needs to be repeated. Also, if a new field is added to the class, every copy of conversion code needs to be updated.
- Code which uses fields of single class, should be part of owner class itself.
- If repetitive code uses fields from multiple classes, or is about things like date formatting, create and use separate Utility classes.
- Understand concurrency. 90% of the cases need simple utilites like [Atomic classes](https://docs.oracle.com/javase/tutorial/essential/concurrency/atomicvars.html) or [Concurrent collections](https://docs.oracle.com/javase/tutorial/essential/concurrency/collections.html) or simply using right scope for variable (method level instead of class level). 
- Keep number of method arguments relatively low (<=4). Once threshold is reached, use criteria/domain POJOs as single encapsulating parameter instead.
- Split large methods and classes into multiple ones. This eases testing (limited dependencies for each task) and encourages code re-use.
- Use ID fields/columns to uniquely identify the record. Avoid using any other fields, especially free-text fields. These may contain special characters (like &) whose meaning changes the context across UI (JS/AJAX), Java and DB.
- [IDE](http://jetbrains.com/idea) is your friend. It is excellent at pointing out inefficient code and ways to resolve it. 

### Spring/Hibernate
- Be careful about enabling Hibernate SQL logging, it is helpful but too verbose. 
I had to update logging config to log only insert/updates, though even that is sometimes too much logging due to Hibernate's SQL Binder.
- Prefer Spring annotations over XML for setup.  
- Avoid writing any logic in controllers (servlets). Controllers should be there only to validate input, call service and return output. 
This helps in testing services without mocking servlet containers, and services can be reused across controllers.
- Avoid JPA interface method names (which auto-generate SQLs). They get too lengthy, write your own HQL queries. Readability is more important that verbosity.

### Terse code
Unfortunately, Java is too verbose. Thus, special care needs to be taken to write succinct code. This helps make business logic more readable (easy to understand).

- If using Java 8 take advantage of [Lambdas](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) and [Streams](https://docs.oracle.com/javase/tutorial/collections/streams/).
- String/Date formatting, collection sorting etc should be done using libraries like [Guava](https://code.google.com/p/guava-libraries/) and [Apache Commons](https://commons.apache.org).
 They provide excellent APIs for routine everyday tasks.
- Annotations are better than XML. In addition, they allow code & corresponding metadata to stay together.

### Processes / Team decisions
- No direct manual changes to production files/codebase. Even minor bug fixes should go through build, tag, deploy process.
- Add version numbers to the UI. This can be a small number in the footer or hidden inside HTML.
- Use automated deployment and rollback processes. I created 200 line Java class to do this. Over time, this effort pays off many times over.
- Backup DB before each deployment. Data is most treasured aspect of a client's ERP software. 
- Create SVN/Git branches for major features. It helps separate production code which might need patching in midst of feature changes.
- Setup continuous integration server if possible. It helps find errors long before you merge branches. 
- Decide as a team if you want implement JavaDocs. If yes, ensure they are in sync with method signatures. I tend to have them only on interfaces used by external modules.
- Comments should be complementary to code. Not a copy of it.
- Use comments only where code is complex or not-easy to read (which itself is a signal to simplify or split code). 
Comments like '// end of if', '// end of for', etc. are nothing but distractions.
- Write comments for documenting domain/business logic, especially for unique/not-obvious use-cases. Eg: Calculations of sales taxes. 

### Performance
- Track [memory load](https://docs.oracle.com/javase/8/docs/technotes/guides/management/jconsole.html) during startup and idle state. 
If memory keeps on increasing, its a red flag. Check if any service is constantly allocating objects. 
- Use pagination instead of loading all data onto UI page. 
- Use Hibernate's findMaxResults or findOne methods to get appropriate results instead of loading all records in memory and choosing few out of them.
- Measure latency for method calls. If using Spring Boot, it can collect these [metrics](http://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html) automatically.
- Don't pre-optimize and overdo caching. Java 8, SQL databases, SSD etc, are incredibly performant. 
I have seen request-db-response flow complete within 100ms without caching. 
- Internal ERP typically don't need sub millisecond latency performance. Focus on software reliability/sturdiness instead.

## Current state of the project
Based on the steps mentioned above, the current state of the project is much healthier.

- Deleted 63,182 lines of code (25% of codebase) while retaining same functionality. Headroom for ~10% more. 
- No unscheduled downtime for server since last 3 months.
- Reduced deployment related errors to zero (although there have been a few deployed-code related bugs). 
- Improved latency by 4x in 2 key functionalities 
- General performance improvement across the board (though not measured, which was my mistake).
- Delivered several feature requests over 5 months (most were small, except one which took 2 months of coding).
- Happy client

### Code snippets of legacy code (a.k.a what to avoid) 
Click on image to enlarge and navigate using swipe/keyboard.

<figure>
    <a href="{{ site.url }}/images/blog/balaji/1-string-format-abuse.png"><img src="{{ site.url }}/images/blog/balaji/1-string-format-abuse.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/2-conversions-to-set-single-field.png"><img src="{{ site.url }}/images/blog/balaji/2-conversions-to-set-single-field.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/3-year-joda-time-repitition.png"><img src="{{ site.url }}/images/blog/balaji/3-year-joda-time-repitition.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/4-string-equals-multiple.png"><img src="{{ site.url }}/images/blog/balaji/4-string-equals-multiple.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/5-method-arguments.png"><img src="{{ site.url }}/images/blog/balaji/5-method-arguments.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/6-date-mess.png"><img src="{{ site.url }}/images/blog/balaji/6-date-mess.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/7-comments-in-if-end.png"><img src="{{ site.url }}/images/blog/balaji/7-comments-in-if-end.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/8-no-code-in-if.png"><img src="{{ site.url }}/images/blog/balaji/8-no-code-in-if.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/9-incorrect-month-and-verbose.png"><img src="{{ site.url }}/images/blog/balaji/9-incorrect-month-and-verbose.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/10-everyline-a-comment.png"><img src="{{ site.url }}/images/blog/balaji/10-everyline-a-comment.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/11-iteration-set-order.png"><img src="{{ site.url }}/images/blog/balaji/11-iteration-set-order.png"></a>
</figure>
<figure>
    <a href="{{ site.url }}/images/blog/balaji/12-actual-code-vs-useless-code.png"><img src="{{ site.url }}/images/blog/balaji/12-actual-code-vs-useless-code.png"></a>
</figure>
