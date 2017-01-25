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
 
I recently started using Spring Boot (1.4) for a new project. 
I learned so much. Its like I have never understood full power of Spring until now. 
I was able to create many orthogonal functionalities (logging, security, auditing etc) 
in the project with minimal amount of code. 

TODO: Create this table at the top, then start the lengthy article.

Spring Env:
- Profiles
- Flyway
- Default - Embedded tomcat, Logging,

Architecture:
- DTO Mappers
- Logging
- Spring auto constructor inject (good for testing)


Spring Security
- Spring User vs Custom User
- Auth with CSRF, CORS, CSP, X-Auth-Token etc built-in
- Encoding passwords - BCrypt
- URL based security
- Method based security
- Update hasAuthority from Role to fine-grained Permissions (custom)
- @CurrentUser (same as AuthenticationPrincipal)

Controllers:
- Exception handling (@RestControllerAdvices)
- DTO Validation
- JSON Conversions (@JSONIgnore)


JPA / Hibernate
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
