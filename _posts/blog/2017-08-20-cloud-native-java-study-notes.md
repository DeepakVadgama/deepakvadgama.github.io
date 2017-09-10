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

## Spring Starter

- Takes care of potentially conflicting libraries that each dependency (eg: JPA, REST, Security etc.) might have.
- Provides Maven/Gradle wrappers such that builds can be reproduce without fear of incompatible versions.
- To build: ```./mvnw clean install```
- To run: ```./mvnw spring-boot:run```
- Spring Boot configures h2 (if present in classpath) and project doesn't contain SQL datasource properties.
- IoC (Inversion of Control) helps with testing (mock injection) and centralizing resource creation & initialization, instead of doing it at call-site (eg: DataSource).
 
## Properties

- Externalize properties. Inject in code using ```@Value = "${property.key:defaultValueIfNotFound}"```
- Spring Boot looks for ```application.properties```
- Choose different name using ```--spring.config.name```
- Default config locations ```classpath:/,classpath:/config/,file:./,file:./config/``` (searched in reverse order)
- Choose different config locations using ```--spring.config.location``` 
- Program arguments with prefix ```--``` (eg: ```--server.port=9090```) are converted to property and added to ```Environment```
- [Excellent Spring Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)

{% highlight shell %}
$ java -jar myproject.jar --spring.config.name=myproject
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
{% endhighlight %}

## Configuration Properties

- String based properties of POJO can be auto-populated with ConfigurationProperties

{% highlight java %}
@SpringBootApplication
@EnableConfigurationProperties
public class ApplicationMain {
    // standard main
}

@ConfigurationProperties(prefix = "mypojo")
public class ConfigProperties {

    private String name;
    private int age;
    private List<String> tags;
  
    // standard getters and setters
}

## in application.properties
mypojo.name=Deepak
mypojo.age=60
mypojo.tags[0]=coding
mypojo.tags[1]=testing

## relaxed binding, all these bind to same property
mail.credentials.auth_method
mail.credentials.auth-method
mail.credentials_AUTH_METHOD
mail.CREDENTIALS_AUTH_METHOD

{% endhighlight %}


## Profiles

- Load environment specific properties using ```application-{profile}.properties``` (eg: ```application-uat.properties```).
- This environment specific property file is loaded on top of ```application.properties``` (properties with same name are overridden).
- Beans can also have profiles (0, 1 or more).
- Beans with no profile are always activated.
- If no profiles are active, ```default``` profile is activated.
- In this case, beans with explicit ```default``` profile value are activated (if any) and ```application-default.properties``` file is loaded (if present).
- ```@Configuration``` classes can also have profiles.
- Activate profile using JVM argument ```-Dspring.profiles.active=dev,hsqldb``` or [any of other variants](http://www.baeldung.com/spring-profiles)
- Environment variables are normalized, and made available as properties (eg: SPRING_PROFILES_ACTIVE is converted to spring.profiles.active).

{% highlight java %}
@Component
@Profile("dev")
public class DevDatasourceConfig {
  // ..
}

@Component
@Profile("!dev")
public class DatasourceConfig {
  // ..
}
{% endhighlight %}

## Spring Cloud Config

### Server

- Externalize configuration on a separate stand-alone repository instead of keeping alongside the code.
- Cloud Foundry provides ConfigServer service to avoid having to manual create/deploy this service.

{% highlight java %}
@SpringBootApplication
@EnableConfigServer
public class ApplicationMain {
    // standard main
}

// in application.properties (better pass as -D argument when starting)
spring.cloud.config.server.git.uri=https://github.com/DeepakVadgama/app1/config-repository
{% endhighlight %}

### Client

- Client application can now get it's config from the server.
- The location of server has to be defined in ```bootstrap.properties``` (or ```bootstrap.yml```).
- This file is loaded before application.properties and tells client where to get the rest of config from.
- Spring Cloud server might have config properties for many applications. Thus each client typically sets a property ```spring.application.name``` in its bootstrap.properties.
- Client also has to set the URI where config server is running, in bootstrap file. Then, on client application start, applications properties are loaded from server (including for active profile).

{% highlight java %}
@SpringBootApplication
public class ApplicationMain {
    // standard main
}

// in bootstrap.properties
spring.application.name=client-application-name
spring.cloud.config.uri=${vcap.services.configuration-service.credentials.uri:http://localhost:8888}
{% endhighlight %}

### Security

Config application can be secured on both server and client using spring security. In this case, on client, the URL is auto-encoded to https://user:pswd@url

{% highlight java %}
spring.cloud.config.username=user
spring.cloud.config.password=pswd
{% endhighlight %}

### Refresh Scope

Refresh scope is a feature to update Spring configuration when there is a property update on config server.
Any component with annotation ```@RefreshScope``` gets its properties refreshed.
There is also a corresponding Spring event ```RefreshScopeRefreshEvent```

{% highlight java %}
@Component
@RefreshScope
public class HelloComponent {

    @Value("${greeting.value}")
    private String greeting;
}

@EventListener
public void refresh(RefreshScopeRefreshEvent event){
}
{% endhighlight %}

