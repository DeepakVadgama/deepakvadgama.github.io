---
layout: post
title: Spring Boot Wonders
category: blog
comments: true
published: false
excerpt: Using Spring Boot 1.4 at its fullest for web applications (with MVC, JPA, Flyway)
tags:
  - development
  - java
  - spring
---
 
Spring Boot is a powerful framework. 
It allows for multitude of functionalities with minimal code (thanks to annotation processing & sensible defaults).
  
Listed below are a few features I've been using in a project. 
If you want to start with pre-configured scaffolded code, checkout [this repository]() 

TODO: Create this table at the top, then start the lengthy article.

## General

### Profiles

Spring Boot allows configuration through [application.properties](https://docs.spring.io/spring-boot/docs/current/.../common-application-properties.html). We can have multiple profiles (application.properties), each for a distinct environment and a developer. Profile can chosen during application start using [vm argument or system property](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html#howto-set-active-spring-profiles) or even set it in supporting IDE like Intellij IDEA.


<figure>
 <a href="{{ site.url }}/images/blog/spring-boot/profiles-full.png"><img src="{{ site.url }}/images/blog/spring-boot/profiles-full.png"></a>
</figure>

<figure style="max-width: 600px; margin-left: auto; margin-right: auto">
 <a href="{{ site.url }}/images/blog/spring-boot/profiles-3.png"><img src="{{ site.url }}/images/blog/spring-boot/profiles-3.png"></a>
</figure>


### Auto constructor injection

Spring Boot [recommends](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3) using constructors for injecting dependencies. If you have single constructor, all the dependencies are auto injected. No need to have @AutoWired or @Inject annotations. As a bonus, this also helps easily mock dependencies in tests. 

{% highlight java %}

// Auto injection
public class FooService {

    private final FooRepository repository;

    public FooService(FooRepository repository) {
        this.repository = repository
    }
}

{% endhighlight %}

### Hot reload of code changes

Spring Boot has fantastic ability to auto-reload changes made to the code. 
If code changes are done within the method, it is auto reloaded. 
Though if there are changes to method parameters, or add/delete of methods/classes, then application needs to be restarted. Even this can be minimized by using [this maven plugin](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-hotswapping.html#howto-reload-springloaded-maven) which restarts the application, and is faster than cold restarts. 
Check out this [documentation](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-hotswapping.html) for more details.

## Spring Security

### Spring User & Authorities

Spring Security requires implementing UserDetailsService, and it uses its own POJO of User (with [limited variables](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/core/userdetails/User.html)). 
If you require more variables you can easily extend the User class and convert instances when required by Spring Security.
 
 
{% highlight java %}

@Service
public class UserService implements UserDetailsService {

    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("Could not find user " + username);
        }
    
        // Convert custom user to spring user
        return new org.springframework.security.core.userdetails.User(
                user.getUsername(), user.getPassword(), user.isEnabled(), true, true,
                true, getAuthorities(user.getRoles()));
    }
}
{% endhighlight %}

### Password encoding

With Spring Security encoding passwords is as easy as writing one line of bean configuration. 
BCrypt is recommended option over MD5 and other hashing algorithms. 

{% highlight java %}

@Bean
public PasswordEncoder encoder() {
    return new BCryptPasswordEncoder(11);
}
user.setPassword(passwordEncoder.encode(userDto.getPassword()));

{% endhighlight %}

### URL based security

URL based security can be very easily setup with few lines of code using WebSecurity configurer. 
 This helps ensure users are authenticated and authorized to access the urls.
 
{% highlight java %}

@Override
protected void configure(HttpSecurity http) throws Exception {

 http
     .authorizeRequests()
        .antMatchers("/","/static/**").permitAll()
        .mvcMatchers("/admin").access("hasAuthority('ROLE_ADMIN')")
        .mvcMatchers("/employees").access("hasAuthority('ROLE_STAFF')")
        .anyRequest().authenticated();
}
 
{% endhighlight %}

### Method level security 


### Spring Expressions 

We can also use Spring expressions 

### Login, Logout, CORS, CSRF, CSP

All the security measures for web-based application is covered by default in Spring Boot. Here's where 
Spring Boot shines with sensible defaults. I highly recommend watching [videos from Spring Security lead 
Rob Winch](https://www.infoq.com/profile/Rob-Winch). 

{% highlight java %}

@Override
protected void configure(HttpSecurity http) throws Exception {

    http
        .authorizeRequests()
            .antMatchers("/","/static/**").permitAll()
            .antMatchers("/test/freeaccess").permitAll()
            .mvcMatchers("/admin").denyAll()
            .anyRequest().authenticated()
            .and()
        .httpBasic()
            .and()
            .csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            .and()
            .formLogin().loginPage("/login").permitAll()
            .and()
            .logout().logoutSuccessUrl("/")
            .and()
        .headers().contentSecurityPolicy("default-src 'self' " +
                                         "https://ajax.googleapis.com " +
                                         "https://cdnjs.cloudfare.com " +
                                         "style-src 'self' 'unsafe-inline' ");

}
    
{% endhighlight %}

 

### AuthenticationPrincipal

- @CurrentUser (same as AuthenticationPrincipal)

## Controllers:

### Input validation

Spring Boot support [JSR 303](http://beanvalidation.org/1.0/spec/)which is bean validation API. 
This can be used to validate inputs. 
It also supports additional hibernate validators, for values to be validated from database (eg: unique email)

{% highlight java %}

public class Person {
    @NotNull
    @Size(min=2, max=30)
    private String name;

    @NotNull
    @Min(18)
    private Integer age;
    
    @NotEmpty 
    @Column(unique = true)
    private String email;
}

@PostMapping("/add")
public String add(@Valid Person person, BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        // handle errors
        return "error";
    }
    return "success";
}

{% endhighlight %}

### Exception handling

When exceptions are not handled in Controllers, they are returned to the caller as exception stack traces. 
Spring Boot supports Global exception handling, where-in we can choose the message and http status to return, 
based on types of exceptions. 

{% highlight java %}
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(value = BadInputException.class)
    public String badRequest(BadInputException e) {
        return e.getMessage();
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler(value = Exception.class)
    public String badRequest(Exception e) {
        return e.getMessage();
    }
}
{% endhighlight %}


### Entity to JSON conversion 

Spring Boot uses Jackson to convert entity/domain objects into JSON for consumption by clients (UI).
There are multiple ways to achieve this:

- Let Spring convert whole class instance (along with its nested hierarchy)
- Use [annotation @JsonIgnore](http://docs.spring.io/spring-data/rest/docs/current/reference/html/#projections-excerpts.projections.hidden-data) to hide some fields from conversion. Eg: for password field, which should not be sent to UI
- Use [JSON views](https://spring.io/blog/2014/12/02/latest-jackson-integration-improvements-in-spring), to contextually convert entities based on requests. 
- Use [separate DTO classes](https://en.wikipedia.org/wiki/Data_transfer_object), and write mappers to convert entity to DTO and vice-versa. DTO pattern is [controversial](https://martinfowler.com/bliki/LocalDTO.html), but the overhead of mappers can be reduced by [BeanUtils.copyProperties()](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/BeanUtils.html)

## Databases / JPA / Hibernate

### Flyway - Database migrations
- ID generation (types = AUTO, IDENTITY, SEQUENCE, TABLE)
http://www.thoughts-on-java.org/jpa-generate-primary-keys/
https://en.wikibooks.org/wiki/Java_Persistence/Identity_and_Sequencing#Identity

- JPA Repository
- Transactional (w/ readyonly)
- Version (for optimistic locking)
- Validation
- Date time, Duration java 8
- CreatedBy, ModifiedBy
- Auditing (w/ Revision repo)
- Create SQL

Testing
- Slices
- Security
- JPA testing (Repo, Auditing etc)
- MockMVC and RestTemplates
- Active Profile
- DirtiesContext
- TestPropertySource
- AutoConfigureTestDatabase + @Sql(mock-users.sql)
- 

## Conclusion

Phew! That's a long list, and there is so much more to explore in Spring Boot. 
I am amazed at how far Java development has come, let alone considering Spring Boot is completely
free and open-source. Go Spring!!


Hit me up in the comments if I missed anything or if you have any queries. 