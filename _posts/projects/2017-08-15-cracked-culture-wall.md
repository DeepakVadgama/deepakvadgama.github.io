---
layout: post
title: Cracked culture wall
category: blog
comments: true
published: false 
excerpt: Working in J.P.Morgan as a Lead Developer
---

Outline
- Spring Boot Migration
- Creating Chatbot for Symphony?
- Internal sessions
- Culture change
- Cloud Foundry

- Migrating to Spring Boot
App with 1.2m requests, load balanced across 4 tomcats
Request params, Servlets + web.xml, 
Huge class for manually inject dependencies

- Culture of complexity
Pass single tomcat, expensive restarts with jpmm having strict SLAs. 
Things change, tomcat shared by only 3 applications (microservices) now

DB change dynamic update can be dangerous. Case in point: ThreadPool replacement on DB change.
There is also a culture of 'what if in the future'

Agile working for us and not other way around. Again manager support very important. 

- Lead developer
I was empowered by my manager's complete support and my position as a lead developer.

- Sessions 
Spring Boot (mostly was ), Testing in Java (link for Github repo), Git (ppt link), Microservices (ppt link)

- Cloud Foundry push
Certification? Migration to CF

- Conclusion
Important to have management support (the world on fire due to DevOps and 'Silicon Vally is coming' scare helps too). This has helped push the agile ideas too. 
