---
layout: post
title: Cloud Native Java Study Notes
category: blog
comments: true
published: false 
excerpt: Notes from Spring/Cloud-Foundry book by Josh Long
tags: 
  - java
  - spring
  - cloud-foundry
---


## Overview

Excellent book. Highly Recommended. 

I predominantly use Spring Boot for my projects. 
Thus, where possible, I've only added Spring Boot way of configuration.
 
Also, I have added topics/items I find interesting from excellent Spring and Cloud-Foundry documentation, which 
are not discussed in the book. 

You might also enjoy 'Spring Boot Wonders'.

## Table of contents


 
## Separate Config from Code

### Properties

- Externalize properties. Inject in code using ```@Value = "${property.key:defaultValueIfNotFound}"```
- Spring Boot looks for ```application.properties```
- Choose different name using ```--spring.config.name```
- Default config locations ```classpath:/,classpath:/config/,file:./,file:./config/``` (searched in reverse order)
- Specify different config locations using ```--spring.config.location``` 
- Program arguments with prefix ```--``` (eg: ```--server.port=9090```) are converted to property and added to ```Environment```
- [Excellent Spring Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)

{% highlight shell %}
$ java -jar myproject.jar --spring.config.name=myproject   ## Choose different name for your properties

$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties   ## Choose property files to load
{% endhighlight %}

### Configuration Properties



### Profiles

- Load environment specific properties using ```application-{profile}.properties``` (eg: ```application-uat.properties```).
- This environment specific property file is loaded on top of ```application.properties``` (properties with same name are overridden).
- Beans can also have profiles (even more than 1). Beans with no profile are always activated. 
- If no profiles are active, ```default``` profile is activated. Beans with ```default``` profile value are activated and ```application-default.properties``` file is loaded (if present). 
- Activate environment using JVM argument ```-Dspring.profiles.active=dev,hsqldb``` or [any of other variants](http://www.baeldung.com/spring-profiles)