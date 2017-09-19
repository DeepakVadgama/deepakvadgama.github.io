---
layout: post
title: Spring Cloud Wonders
category: blog
comments: true
published: true
excerpt: Spring features useful for building Cloud Native applications
tags: 
  - java
  - spring
  - spring-cloud
---

Over the past decade, Spring has become de facto standard in large enterprises for creating all kinds of applications.
Unsurprisingly, my previous post on [Spring Boot]({{site.url}}/post/spring-boot-wonders) is the most popular article on this blog. In this post I've attempted to detail some additional aspects of Spring from perspective of creating Cloud Native applications.


## Table of contents

- [Spring Starter](#spring-starter)
- [Properties](#properties)
- [Configuration Properties](#configuration-properties)
- [Profiles](#profiles)
- [Spring Cloud Config](#spring-cloud-config)
  * [Server](#server)
  * [Client](#client)
  * [Security](#security)
  * [Refresh Scope](#refresh-scope)
- [Session Replication](#session-replication)
- [Async Controller](#async-controller)
- [Async Service](#async-service)
- [Service Discovery](#service-discovery)
- [Resources](#resources)
- [Conclusion](#conclusion)


## Spring Starter

- Takes care of potentially conflicting libraries that each dependency (eg: JPA, REST, Security etc.) might have.
- Provides Maven/Gradle wrappers such that builds can be reproduce without fear of incompatible versions.
- To build: ```./mvnw clean install```
- To run: ```./mvnw spring-boot:run```
- Spring Boot configures h2 (if present in classpath) and project doesn't contain SQL datasource properties.
- IoC (Inversion of Control) helps with testing (mock injection) and centralizing resource creation & initialization, instead of doing it at call-site (eg: DataSource).

<figure>
    <a href="{{ site.url }}/images/blog/spring-boot/spring-starter.png">
   <img src="{{ site.url }}/images/blog/spring-boot/spring-starter.png">
    </a>
</figure>

## Properties

- Accessible by [Website](https://start.spring.io), [IDE](https://www.jetbrains.com/help/idea/2017.1/creating-spring-boot-projects.html) or [Command Line](https://github.com/spring-io/initializr#generating-a-project)
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

// in application.properties
mypojo.name=Deepak
mypojo.age=60
mypojo.tags[0]=coding
mypojo.tags[1]=testing

// relaxed binding, all these bind to same property
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

## Session Replication

Spring Session helps with session replication by replacing Servlet HTTP Session API and storing the session information in Redis/Hazelcast etc.
Spring Boot makes this configuration dead simple.
Read more about it [here](https://spring.io/blog/2015/03/01/the-portable-cloud-ready-http-session)

{% highlight xml %}
// application.properties
spring.session.store-type=redis
server.session.timeout=5

// pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
{% endhighlight %}

## Async Controller

Spring MVC (or any servlet container) creates a thread-pool to handle servlet requests.
There is still a possibility of thread being occupied for too long by some expensive operation.
Simply wrapping the response in a ```Callable``` makes the code run in a separate thread-pool (can be overridden by creating a TaskExecutor bean) there by
freeing the servlet thread-pool to accept more requests.

{% highlight java %}
@Controller
public class MyController {

    @RequestMapping("/username")
    public @ResponseBody Callable<String> getUsername() {
        return () -> {
                // time-consuming operation
                return getUsername();
            }
        };
    }
}
{% endhighlight %}

## Async Service

Same concept can be extended to service calls by using ```@Async``` and Java's ```CompletableFuture```.

{% highlight java %}
@SpringBootApplication
@EnableAsync
public class ApplicationMain {
    // standard main
}

@Service
public class SearchService {

	@Async
	public Future<SearchResult> search(String keyword) {
		SearchResult result = ... // time-consuming operation
		return new AsyncResult<SearchResult>(result);
	}
}
{% endhighlight %}

## Service Discovery

- Service discovery is the life line of a microservices based application.
- It typically contains a well-known registry service. Components use this service to register themselves and discover other components.
- Recommended to use ```spring.application.name``` for all client components to allow registry/discovery using logical names.
- [Spring article](https://spring.io/guides/gs/service-registration-and-discovery/)

{% highlight java %}

// server
@EnableEurekaServer
@SpringBootApplication
public class ServerApplication {
    // standard main
}

// server application.properties
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

// client
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaClientApplication {
    // standard main
}

@RestController
class ServiceInstanceRestController {

    @Autowired
    private DiscoveryClient discoveryClient;

    public void validateInstance() {
        Assert.notNull(discoveryClient.getInstances("dependent-service-id"));
    }
}
// client bootstrap.properties
spring.application.name=user-service
eureka.instance.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
{% endhighlight %}

## Resources

I used the following resources to write this article.

- [Cloud Native Java](http://shop.oreilly.com/product/0636920038252.do)
- [Spring Cloud Fundamentals - Pluralsight](https://app.pluralsight.com/library/courses/spring-cloud-fundamentals/table-of-contents)
- [Spring Cloud Documentation](http://projects.spring.io/spring-cloud/)

## Conclusion

This post covers only a part of what Spring Cloud has to offer. It offers
 [Circuit Breakers](https://spring.io/guides/gs/circuit-breaker/),
 [Load Balancers](https://cloud.spring.io/spring-cloud-netflix/),
 [Spring Cloud Security](http://cloud.spring.io/spring-cloud-security/),
 [Event Messaging](https://reflectoring.io/event-messaging-with-spring-boot-and-rabbitmq/) and so much more.
 I intend to write more on rest of the topics as soon as I have some hands-on experience.

 Hit me up in the comments if I missed anything or if you have any queries.
