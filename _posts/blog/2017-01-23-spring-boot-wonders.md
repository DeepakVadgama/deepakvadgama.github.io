---
layout: post
title: Spring Boot Wonders
category: blog
comments: true
excerpt: Using Spring Boot 1.4 at its fullest for web applications (with MVC, JPA, Flyway)
tags:
  - development
  - java
  - spring
---
 
Spring Boot is mighty powerful. It enables multitude of functionalities with minimal amount of code (thanks to annotation processing & sensible defaults).
  
Listed below are a few features I've been using in a new project. I am creating a pre-configured scaffold code repository, will share the same soon. 

- [General](#general)
  * [Profiles](#profiles)
  * [Dependency injection without annotations](#dependency-injection-without-annotations)
  * [Hot reload of code changes](#hot-reload-of-code-changes)
- [Spring Security](#spring-security)
  * [Spring User & Roles](#spring-user---roles)
  * [Password encoding](#password-encoding)
  * [URL based security](#url-based-security)
  * [Method level security](#method-level-security)
  * [Spring Expressions based authorization](#spring-expressions-based-authorization)
  * [Login, Logout, CORS, CSRF, CSP](#login--logout--cors--csrf--csp)
  * [Accessing current user](#accessing-current-user)
- [Controllers](#controllers)
  * [Input validation](#input-validation)
  * [Exception handling](#exception-handling)
  * [Entity to JSON conversion](#entity-to-json-conversion)
- [Databases / JPA / Hibernate](#databases---jpa---hibernate)
  * [General](#general-1)
  * [Flyway - Database migrations](#flyway---database-migrations)
  * [Primary key](#primary-key)
  * [Java 8 date, time and duration](#java-8-date--time-and-duration)
  * [Auditing record metadata](#auditing-record-metadata)
  * [Auditing full records](#auditing-full-records)
  * [SQL creation](#sql-creation)
- [Testing](#testing)
  * [General](#general-2)
  * [Slices](#slices)
  * [Security](#security)
  * [JPA testing](#jpa-testing)
  * [MockMVC and RestTemplates](#mockmvc-and-resttemplates)
- [Conclusion](#conclusion)


## General

### Profiles

Spring Boot allows configuration through [application.properties](https://docs.spring.io/spring-boot/docs/current/.../common-application-properties.html). We can have multiple profiles (application.properties), each for a distinct environment and a developer. Specific profile can chosen during application startup using [vm argument or system property](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html#howto-set-active-spring-profiles) or even set it in supporting IDE like Intellij IDEA.


<figure>
 <a href="{{ site.url }}/images/blog/spring-boot/profiles-full.png"><img src="{{ site.url }}/images/blog/spring-boot/profiles-full.png"></a>
</figure>

<figure style="max-width: 600px; margin-left: auto; margin-right: auto">
 <a href="{{ site.url }}/images/blog/spring-boot/profiles-3.png"><img src="{{ site.url }}/images/blog/spring-boot/profiles-3.png"></a>
</figure>


### Dependency injection without annotations

Spring Boot [recommends](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3) using constructors for injecting dependencies. If you have single constructor, all the dependencies are auto injected. No need to have @AutoWired or @Inject annotations. As a bonus, this also helps mocking dependencies in tests (without using Spring constructs). 

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

Spring Boot has fantastic ability to auto-reload changes made to the code without restarting the server. 
If code changes are done within the method, that code snippet is auto reloaded. 
Though if there are changes to method parameters, or add/delete of methods/classes, then application needs to be restarted. Even this can be minimized by using [this maven plugin](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-hotswapping.html#howto-reload-springloaded-maven) which detects changes and restarts the application. It is faster than 'cold' restarts. Check out this [documentation](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-hotswapping.html) for more details.

## Spring Security

### Spring User & Roles

Spring Security requires implementing UserDetailsService, and it uses its own POJO of User (with [limited variables](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/core/userdetails/User.html)). 
If you require more variables you can easily extend the User class and convert instances when required by Spring Security.
 Side note: In Spring Security terminology a user-role is called Authority.
 
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

With Spring Security, encrypting passwords (to be persisted in DB), is as easy as writing one line of bean configuration. 
BCrypt is [recommended](https://en.wikipedia.org/wiki/Bcrypt) option over MD5 and other hashing algorithms. 

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

@Configuration
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
    
     http
         .authorizeRequests()
            .antMatchers("/","/static/**").permitAll()
            .mvcMatchers("/admin").access("hasAuthority('ROLE_ADMIN')")
            .mvcMatchers("/employees").access("hasAuthority('ROLE_STAFF')")
            .anyRequest().authenticated();
    }
}
 
{% endhighlight %}

### Method level security 

Individual methods of controllers/services can be restricted too.  
 
{% highlight java %}

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

}

@RestController 
@PreAuthorize("hasAuthority('READ_MACHINE')")   // Applies to all methods
public class MachineController {
    
    @GetMapping("/id/{machineId}")
    public MachineDto getById(@PathVariable Long machineId) {
        return machineMapper.toDto(machineService.getById(machineId));
    }
    @PostMapping("/delete/{id}")
    @PreAuthorize("hasAuthority('DELETE_MACHINE')")
    public boolean delete(@PathVariable Long id) {
        return machineService.delete(id);
    }
}
{% endhighlight %}

### Spring Expressions based authorization

These restrictions need not be just based on authorities. With use of [Spring Expressions](https://docs.spring.io/spring/docs/current/spring-framework.../html/expressions.html) 
it can trigger any service method, access parameters, access return value, access a bean etc. This can be useful for requests which depend on data, or where authorization code is dynamic and is implemented in separate service. 

{% highlight java %}
public class IBookService {
	@PreAuthorize ("hasRole('ROLE_WRITE')")
	public void addBook(Book book){
	    //..
	}

	@PostAuthorize ("returnObject.owner == authentication.name")
	public Book getBook(){
	    //.. 
	}

	@PreAuthorize ("#book.owner == authentication.name")
	public void deleteBook(Book book){
	    //..
	}
}
{% endhighlight %}

### Login, Logout, CORS, CSRF, CSP

All the security measures for web-based application are covered by default in Spring Boot. Here's where 
Spring Boot shines with sensible defaults. I highly recommend watching [videos from Spring Security lead 
Rob Winch](https://www.infoq.com/profile/Rob-Winch). 

{% highlight java %}

@Override
protected void configure(HttpSecurity http) throws Exception {

    http
        .authorizeRequests()
            .antMatchers("/","/static/**").permitAll()
            .mvcMatchers("/admin").access("hasAuthority('ROLE_ADMIN')")
            .mvcMatchers("/employees").access("hasAuthority('ROLE_STAFF')")
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


### Accessing current user

Spring can tell which user the request belongs to. This can be achieved by only using annotation 
@AuthenticationPrincipal with the method parameter (of type User). If @AuthenticationPrincipal is not very readable, then custom annotation can be created which does the same thing. 

{% highlight java %}

// Controller method
@PostMapping("/edit/password/self")
public void updatePassword(@RequestParam String updatedPassword, @AuthenticationPrincipal User user) {
    userService.updatePassword(user.getUsername(), updatedPassword);
}

OR 

// Custom annotation class for AuthenticationPrincipal
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@AuthenticationPrincipal
public @interface CurrentUser {

}

// Controller method
@PostMapping("/edit/password/self")
public void updatePassword(@RequestParam String updatedPassword, @CurrentUser User user) {
    userService.updatePassword(user.getUsername(), updatedPassword);
}

{% endhighlight %}

## Controllers

### Input validation

Spring Boot supports [JSR 303](http://beanvalidation.org/1.0/spec/) bean validation API. 
This is very useful in validating user inputs. 
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
based on type of exceptions. 

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

- Let Spring convert whole class instance (along with its nested hierarchy) into JSON
- Use [annotation @JsonIgnore](http://docs.spring.io/spring-data/rest/docs/current/reference/html/#projections-excerpts.projections.hidden-data) to hide some fields from conversion. Eg: for password field, which should not be sent to UI
- Use [JSON views](https://spring.io/blog/2014/12/02/latest-jackson-integration-improvements-in-spring), to contextually convert entities based on requests 
- Use [separate DTO classes](https://en.wikipedia.org/wiki/Data_transfer_object), and write mappers to convert entity to DTO and vice-versa. DTO pattern is [controversial](https://martinfowler.com/bliki/LocalDTO.html), but the overhead of mappers can be reduced by [BeanUtils.copyProperties()](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/BeanUtils.html)

## Databases / JPA / Hibernate

### General

Spring Boot (and Spring Data JPA) allows for lot of basic functionalities with minimal code: 

- @Column and @Table annotation for entity is optional, naming strategies have [sensible defaults](https://docs.jboss.org/hibernate/orm/5.1/userguide/html_single/chapters/domain/naming.html). 
- Lazy fetching of nested object [hierarchy](https://dzone.com/articles/jpa-lazy-loading)
- Field validations as mentioned above
- [DDL validation](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html) during startup
- Transactions with annotations [@transactional and @transactional(readonly = true)](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#transactions)
- JPA Repositories for [CRUD and Paging/Sorting](https://docs.spring.io/spring-data/data-commons/docs/1.6.1.RELEASE/reference/html/repositories.html)
- Optimistic locking with [single field/annotation of @version](http://www.byteslounge.com/tutorials/jpa-entity-versioning-version-and-optimistic-locking)

### Flyway - Database migrations

We have ample resources for deployments and rollback of the code, but very limited tools for doing the same
for database changes done during the release. [Flyway](https://flywaydb.org/) is an excellent tool which does just that.
Spring Boot integrates nicely with Flyway. 

Check out [this page](https://flywaydb.org/getstarted/how) for how flyway works. It is quite useful and effective
for DB upgrades and rollbacks. 
 
 Check out [this sample repository](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html)
 
 <figure style="max-width: 300px" >
  <a href="{{ site.url }}/images/blog/spring-boot/flyway.png"><img src="{{ site.url }}/images/blog/spring-boot/flyway.png"></a>
 </figure> 

### Primary key
 
 Primary key for records can be generated using one of these types (IDENTITY, SEQUENCE, TABLE).
 Each has pros and cons. For most use cases AUTO will do Check out [more details here](http://www.thoughts-on-java.org/jpa-generate-primary-keys/) and [here](https://en.wikibooks.org/wiki/Java_Persistence/Identity_and_Sequencing#Identity)
The default value for @GeneratedValue is AUTO. 

{% highlight java %}
    @Entity
    public class User {
    
        @Id
        @GeneratedValue
        private Long id;
    }
{% endhighlight %}

### Java 8 date, time and duration

Earlier Joda Date (& Java 8 date) had to be converted to java.sql.Date to persist into DB. 
This manual conversion can be avoided by adding only a converter in configuration. 
Now all entities can use Java 8's Date, Time and Duration classes and the values are stored and retrieved
correctly from database. 

{% highlight java %}
@EntityScan(basePackageClasses = {MyApplication.class, Jsr310JpaConverters.class})
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MachineMaintenanceApplication.class, args);
    }
}
{% endhighlight %}

### Auditing record metadata

Auditing the metadata of creates & updates is supported out of the box. 

- Using @CreatedDate and @ModifiedDate annotations on entity fields is enough for those columns to be 
 populated each time a record is created or updated. 
- Using @CreatedBy and @ModifiedBy annotations allow to insert user who created/updated the record. For this, 
Spring needs to know who is the auditor. This auditor can be configured with few lines of code.
- The modified-date and modified-by fields only store data related to latest update. 

{% highlight java %}

@Entity
@EntityListeners(AuditingEntityListener.class)
public class Person {
    
    @Column(columnDefinition = "DATETIME", nullable = false, updatable = false)
    @CreatedDate
    protected LocalDateTime createdDate;
    
    @Column(columnDefinition = "DATETIME")
    @LastModifiedDate
    protected LocalDateTime modifiedDate;
    
    @CreatedBy
    protected String createdBy;
    
    @LastModifiedBy
    protected String modifiedBy;
}

// Auditor Config
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class PersistenceConfig {

    @Bean
    AuditorAware<String> auditorProvider() {
        return new AuditorAwareImpl();
    }

    // This class is used for Spring JPA entities to get & populate createdBy and modifiedBy username strings
    private class AuditorAwareImpl implements AuditorAware<String> {
        @Override
        public String getCurrentAuditor() {
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            if (authentication == null || !authentication.isAuthenticated()) {
                return null;
            }
            return ((User) authentication.getPrincipal()).getUsername();
        }
    }
}

{% endhighlight %}

### Auditing full records

[Hibernate envers](hibernate.org/orm/envers/) allows to audit entire row of the database each time it is updated. It can be easily used in Spring Boot by using a lightweight wrapper project called [Spring-data-envers](https://github.com/spring-projects/spring-data-envers). The auditing comes extremely handy. The database can be just 
queried for changes instead of digging through the logs (or using log parsers). The audited records can also be accessed programmatically by extending RevisionRepository. 

{% highlight java %}
@Entity
@Audited
public class Person {
    //...fields...    
}

public interface PersonRepository extends RevisionRepository<Machine, Long, Integer>, JpaRepository<Machine, Long> {
    //..extra methods.. 
}

@Configuration
@EnableJpaRepositories(repositoryFactoryBeanClass = EnversRevisionRepositoryFactoryBean.class)
public class PersistenceConfig {
  //.... 
}
{% endhighlight %}

### SQL creation

Hibernate can [create the missing tables on application startup](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html) (based on the entity classes in the application). If [Flyway DB](https://flywaydb.org/) is being used, we need to have create/update queries (DDL) in flyway SQL scripts. These queries need not be written manually. They can be generated by Hibernate on application startup. Once configured, all the DDL queries (along with constraints, indexes and joins) are dumped into the specific file from which we can copy-paste to Flyway SQL scripts file.

{% highlight properties %}
spring.jpa.properties.javax.persistence.schema-generation.create-source=metadata
spring.jpa.properties.javax.persistence.schema-generation.scripts.action=create
spring.jpa.properties.javax.persistence.schema-generation.scripts.create-target=create.sql
{% endhighlight %}

<figure style="max-width: 600px">
 <a href="{{ site.url }}/images/blog/spring-boot/create-sql.png"><img src="{{ site.url }}/images/blog/spring-boot/create-sql.png"></a>
</figure>

## Testing

### General

- @ActiveProfile: Chooses the profile to be used (for all tests in that class)
- @DirtiesContext: Resets context after every test
- @TestPropertySource: Adds extra test properties configuration
- @ComponentScan: Scan only specific package classes to speeden up the process

### Slices
 
 Spring Boot allows to test each individual layer (DAO, Controller etc) separately. 
 Each layer has corresponding annotations (@WebMvcTest, @DataJpaTest) to be used. 
 This helps in fast execution of tests, (initialization of layers unrelated to the tests
 are be avoided), also making code more succinct. Check [this article](https://www.google.co.in/search?client=safari&rls=en&q=spring+boot+1.4+testing&ie=UTF-8&oe=UTF-8&gfe_rd=cr&ei=5z2KWPvrJtP08wft55qABA) for more details. 
  
### Security

If application is configured to use authentication and authorization, users (for the requests) can be configured per test-class or per test-method. Also, user can be set with specific authorities, or an existing DB user can be used.

{% highlight java %}
@Sql("classpath:mock-user.sql")   // Insert test data
@WithUserDetails("user1")     // Run all methods of class with user1
public class AuditingTests {
    
    @Test
    @WithAnonymousUser   // Run with anonymous user
    public void testUnAuthenticated() throws Exception {
        mvc.perform(get("/test/freeaccess"))
                .andExpect(status().isOk())
                .andExpect(content().string("Working"));
    }

    @Test
    @WithMockUser(username = "testUser", authorities = {"ANY_PERMISSION"})
    public void testAuthenticatedSuccess() throws Exception {
        mvc.perform(get("/test/authenticated"))
                .andExpect(status().isOk())
                .andExpect(content().string("Authenticated"));
    }
}
{% endhighlight %}


### JPA testing 

During JPA testing, 

- @DataJpaTest can be used for testing just JPA layer
- @AutoConfigureTestDatabase can be used to auto-configure test database (requires H2 or similar DB in classpath)
- @Sql can be used to insert mock data before tests run
- @EnableJpaAuditing can be used to well, enable auditing
- @AutoConfigureTestEntityManager can be used to auto-configure entity manager for persisting entities


{% highlight java %}
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureTestEntityManager
@AutoConfigureTestDatabase
@EnableJpaAuditing
@Sql("classpath:mock-user.sql")
public class AuditingTests {

}
{% endhighlight %}

### MockMVC and RestTemplates

The controllers can be tested using either mock-mvc or with rest templates. MockMVC as name suggests only mocks the network calls. With rest-templates, server needs to be started and [network calls are made](http://stackoverflow.com/a/25902063/3494368) for testing.  
 
 {% highlight java %}
 @RunWith(SpringRunner.class)
 @SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
 public class MyTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    public void test() {
        this.restTemplate.getForEntity(
            "/{username}/vehicle", String.class, "Phil");
    }
 }
 
 OR
 
 @RunWith(SpringRunner.class)
 @SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
 @TestPropertySource(locations = "classpath:test.properties")
 @AutoConfigureTestDatabase
 public class MachineIntegrationTests {
 
     @Autowired
     private WebApplicationContext context;
 
     private MockMvc mvc;
 
     @Before
     public void setup() {
         mvc = MockMvcBuilders
                 .webAppContextSetup(context)
                 .apply(springSecurity())
                 .build();
     }
    
     @Test
     public void testUnAuthenticated() throws Exception {
         mvc.perform(get("/test/freeaccess"))
                 .andExpect(status().isOk())
                 .andExpect(content().string("Working"));
     }
 }
 {% endhighlight %}

## Conclusion

Phew! That's a long list, and yet it does not cover all the features Spring Boot has to offer (eg: [Actuators and metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html)). It amazes me that Java development has come this far. I still remember reinventing the wheel (& writing too much code for it) for every project, not to mention the testing required. This is even more amazing considering Spring Boot (& family) is completely free and open-source. Good times!

Hit me up in the comments if I missed anything or if you have any queries. 
