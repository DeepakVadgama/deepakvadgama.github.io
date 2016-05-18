---
layout: post
title: Redeploy WAR to Tomcat using Maven
category: blog
comments: true
excerpt: Save time during WAR redeploys to Tomcat using Maven
tags: tomcat, maven
---

I have been working on an existing Spring MVC application with relatively large codebase. 
It became important to reduce the time taken to redeploy the WAR with code change. There is an easy though multi-step solution for this. 
This article details these steps. There are also multiple alternatives to this flow, listed at the end of the article. 
  
Steps for single step 'compile, build war, deploy and reload' war to Tomcat using Maven:  

### Tomcat Manager
Tomcat comes bundled with web-app called [manager](https://tomcat.apache.org/tomcat-8.0-doc/manager-howto.html) which allows us to check and deploy applications to Tomcat using URL. 
Ensure that tomcat/webapps/manager directory is present. If not available, download fresh version of Tomcat from [here]()

### Setup Tomcat User
For security reasons by default users do not have access to Manager application. 
To gain access add a user in tomcat/conf/tomcat-users.xml file. Update username/password to your liking 

{% highlight xml %}
<user username="tomcat" password="tomcat" roles="admin,manager,manager-gui,manager-script"/>
{% endhighlight%}


### Add same credentials to Maven
Since we will use maven to deploy to tomcat and we are not going to hardcode the username-password in our pom.xml, 
let us create a server tag in maven/conf/settings.xml file (do ensure IDE is using same maven settings xml). 
Ensure same username, password as above. Change id value to you liking. 
    
{% highlight xml %}
<servers>
  <server>
    <id>Tomcat</id>
    <username>tomcat</username>
    <password>tomcat</password>
  </server>
</servers>
{% endhighlight%}

### Setup tomcat maven plugin
Add tomcat maven plugin to your application's pom.xml. 
This will allow us to trigger manager url (localhost:8080/manager) to deploy our war. 
Update context path (/salesmanager) based on your application, and server id defined in maven's setting.xml.
context path is typically same as name of your WAR.
A common mistake is to forget the 'text' string after the url, or use html instead. Be careful.  
  
{% highlight xml %}
 <plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <url>http://localhost:8080/manager/text</url>
        <path>/salesmanager</path>
        <server>Tomcat</server>
    </configuration>
 </plugin>
{% endhighlight %}

### Run tomcat:deploy task

Since we added this new plugin, your IDe should download this automatically and show corresponding tasks available.
Ensure tomcat is started and the run task 'tomcat:redeploy' from Maven Projects window and check console for corresponding status

 <figure>
     <a href="{{ site.url }}/images/blog/maven-tasks-tomcat.png"><img src="{{ site.url }}/images/blog/maven-tasks-tomcat.png"></a>
 </figure>
 
  <figure>
      <a href="{{ site.url }}/images/blog/maven-tomcat-redeploy.png"><img src="{{ site.url }}/images/blog/maven-tomcat-redeploy.png"></a>
  </figure>

If all goes well, WAR will be redeployed and will be reflected on browser http://localhost:8080/you-context-path; in this case http://localhost:8080/salesmanager 

For good measure add shortcut to this tomcat task to avoid opening Maven task window, finding and clicking the same task repeatedly. 


### WAR size limit
You might face FileSizeLimitExceeded exception while deploying the application. 
This is because WAR size is greated than acceptable limit. 
There are 2 reasons this might happen either http post doesn't allow it or manager web-app is refusing.

Update maxSwallowSize and connectionTimeout in tomcat/conf/server.xml 

{% highlight xml %}
 <Connector port="8080" protocol="HTTP/1.1"
            connectionTimeout="20000"
            maxSwallowSize="100000000"
            redirectPort="8443" />
{% endhighlight%}

Update/Validate file size limit in tomcat/webapps/manager/WEB-INF/web.xml. 
Note this step is only if you are using html manager interface (http://localhost:8080/manager/text), while we are using text interface (http://localhost:8080/manager/text). 

{% highlight xml %}
 <multipart-config>
   <!-- 80MB max -->
   <max-file-size>80428800</max-file-size>
   <max-request-size>80428800</max-request-size>
   <file-size-threshold>0</file-size-threshold>
 </multipart-config>
{% endhighlight%}


### ArrayIndexOutOfBoundsException while reading ______$1.class

This issue occurs when the JDK version the class is compiled with is different than the compatible Spring and Tomcat versions.

Spring 3.x doesn't support JDK 8.x. Avoid downgrading JDK to 7 because it is no longer supported, thus might have security loop holes. 
Better yet, upgrade to Spring 4.x using [this guide](https://github.com/spring-projects/spring-framework/wiki/migrating-from-earlier-versions-of-the-spring-framework)

Also validate your tomcat version based on [this chart](http://tomcat.apache.org/whichversion.html)


## Alternatives

### Spring boot 
Spring Boot v1.3 supports [Auto-reload and live-server](https://spring.io/blog/2015/06/17/devtools-in-spring-boot-1-3) which is better than 
having to manually trigger the deploys.  
  
### JRebel
[JRebel](https://zeroturnaround.com/software/jrebel/) is a tool which supports hot-swap of classes and other resources. It supports multitude of 
frameworks like Spring, Hibernate etc and in the long run can save a lot of time avoiding application restarts. 
Personally I feel its way too expensive ($475 per user per year) even though it is useful. 
Alternative to this is [DCEVM](http://ssw.jku.at/dcevm/), but its no longer being actively developed and it does not have support for various frameworks
 and libraries. 
 
### Intellij Idea / Eclipse hot swap
IDEs support hotswap of files and combined with Tomcat's ability to watch and reload updated files, it provides good alternative to buying JRebel. 
Go through [Intellij Idea setup](http://stackoverflow.com/a/19609115/3494368) and [Eclipse setup](http://www.mkyong.com/eclipse/how-to-configure-hot-deploy-in-eclipse/). 
This alternative though works only for changes within a method and is not extensive as JRebel. Nonetheless very useful. 

### Tomcat Context Reload
Tomcat allows reloading context (without restarting the application). For this add the following to Tomcat's conf/server.xml file (within existing host tag)
 
 {% highlight xml %}
 <Host>
    <Context path="/path-dir-containing-war" reloadable="true">
        <WatchedResource>path/to/watched/resource</WatchedResource>
        <WatchedResource>another/path/to/another/resource</WatchedResource>
    </Context>
 </Host>
 {% endhighlight %}
 
 
That's that. Enjoy the redeploys and comment below if you need help with any of the steps.  