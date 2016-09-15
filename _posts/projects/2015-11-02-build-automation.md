---
layout: post
title: Build Automation
category: projects
comments: true
excerpt: Build automation of microservices based application using Bamboo API   
---

In one of my projects based on a trading system, we converted a monolith into microservices. I talked about it in [my earlier post](http://deepakvadgama.com/projects/accidental-microservices/). Microservices helped us become a more efficient team, though the supporting build system wasn’t able to keep up. 

## Scope
Each microservice had multiple dependencies; a combination of third-party JARs and internal projects (Caching, Transport, POJO etc). There were more than 30 projects to be built, making up total of ~18 microservices.

## Dependency mess
The messy build hierarchy with criss-crossing dependencies made it even more cumbersome. The picture below is a snippet of the build tree; the complete tree was much deeper and broader. Few of dependencies at the base level (eg: those containing Trade POJO) were used by most of the projects. Meaning, if those changed, whole tree had to be rebuilt.

<figure>
    <a href="/images/blog/build_hierarchy.png"><img src="/images/blog/build_hierarchy.png"></a>
</figure>

## Manual update of pom.xml
The existing build process was very manual

+ Trigger build of base project(s) using [Bamboo](https://www.atlassian.com/software/bamboo/).
+ Get resulting version of the build artifact.
+ Update the version of dependency in all parent projects' pom.xmls.
+ Trigger build for parent project(s).
+ Rinse and repeat (bottom-up).

It took us ~4 hours to build the whole system due to the scope, and this manual process. Not to mention frequent human errors.

## Solution
I was lucky to be assigned the task to resolve this issue. After some research we landed on Java based solution which was composed of the following

## Bamboo
We had to prepare each project in Bamboo for the automation. We created 2 stages for each build: Release and Snapshot. Snapshot was nothing special (since it did not involve updating dependencies). Release job was made up of many tasks: Source Code Checkout, Maven plugin tasks (detailed in next sections), Maven Release etc. 

<figure>
    <a href="/images/blog/bamboo_tasks.png"><img src="/images/blog/bamboo_tasks.png"></a>
</figure>

## Bamboo API
[Bamboo REST API](https://developer.atlassian.com/bamboodev/rest-apis/bamboo-rest-resources) is a fantastic resource to create custom build solutions. Though we used it only to trigger builds and monitor results, the APIs are extensive enough to do analytics of the build process. 

+ [API to trigger build using Project and Build keys](https://docs.atlassian.com/bamboo/REST/5.9.7/#d2e82)
+ [API to monitor the progress (called every 5 seconds)](https://docs.atlassian.com/bamboo/REST/5.9.7/#d2e385)

## SSO for Bamboo REST API
For monitoring purposes the Bamboo REST API was authenticated using SSO (Single Sign On), which was transparent when using browsers (thanks to cookies). This would not be transparent when triggered through Java. 

One solution was to request for a new SSO system account to use. I skipped this option. The turnaround time of getting this account was high, and we would’ve lost ability to track who triggered respective builds. 

We used [Apache HTTP libraries](http://hc.apache.org/) to simulate the SSO. So anyone triggering the build (using Java) would enter the SSO credentials in the console (replacing clear-text passwords in console with asterixis ****** is [hard](http://stackoverflow.com/questions/15159762/how-to-read-password-from-console-without-using-system-console)). The [JSESSIONID](http://www.cs-repository.info/2014/07/understanding-jsessionid.html) received in response, was stored and used in subsequent Bamboo REST calls thus solving SSO issue.

## Bamboo parent-child relationship
The manual step of updating the child dependency’s new version in parent pom.xml had to be fixed. We had an option of using Maven's feature of using LATEST version available in central repository. This imposed constraint of always using the latest version and was not flexible. We found [Versions Maven plugin](http://www.mojohaus.org/versions-maven-plugin/) which does the exact same task, but offers lot of flexibility:

+ [use-latest-releases](http://www.mojohaus.org/versions-maven-plugin/use-latest-releases-mojo.html): To update to latest releases
+ [use-releases](http://www.mojohaus.org/versions-maven-plugin/use-releases-mojo.html): To replace snapshots with release versions.
+ [includesList](http://www.mojohaus.org/versions-maven-plugin/use-latest-releases-mojo.html#includesList): Update only dependencies mentioned in the list passed as parameter at runtime. 

The changes made by this plugin are done in local checked-out files. We used [SCM plugin](https://maven.apache.org/scm/maven-scm-plugin/checkin-mojo.html) to check-in the updated pom.xmls to SVN repository. Each plugin was configured using Bamboo's Maven task shown below:
 
<figure>
    <a href="/images/blog/bamboo_maventask.png"><img src="/images/blog/bamboo_maventask.png"></a>
</figure>

## Tree for each service
We created a single Java class which could trigger the whole build of application. We also created multiple classes each building a microservice. This was obvious option now that each microservice had a distinct owner/team working on it.

## Build monitoring and results
This Java class used Bamboo API detailed above to trigger builds, retrieve the build number. Monitor the progress using the build number. This java code also had a list of all dependencies which were passed as parameters to the Maven plugins detailed above. Thus, skipping an update of dependency update was as simple as commenting it out from the list.

<figure>
    <a href="/images/blog/bamboo_build_queue.png"><img src="/images/blog/bamboo_build_queue.png"></a>
</figure>

<figure>
    <a href="/images/blog/bamboo_results.png"><img src="/images/blog/bamboo_results.png"></a>
</figure>

## Results
This task took around 2 months; much longer than I anticipated (it involved lot of research). In the end, it was an exciting and very satisfying challenge. Whole team started using the product immediately; no more error prone repetitive builds. Best of all, it saved us at least 2 days for every sprint (which was typically 2 weeks).   