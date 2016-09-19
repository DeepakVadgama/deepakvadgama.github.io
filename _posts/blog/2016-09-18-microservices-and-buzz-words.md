---
layout: post
title: Microservices, Docker and buzz words
category: blog
comments: true
excerpt: Decoding the buzz words related to microservices ecosystem
tags: 
  - cloud
  - microservices
---

Can’t make sense of words like Docker, Containers, Kubernetes, ETCD? You are not alone. The container technology space is on a tear lately. Let me try to make sense of a few the buzz words. 

**Caution**: I only have limited understanding of this ecosystem. I’d suggest reading the article to understand concepts rather than individual companies/projects and where they stand.

## Microservices
Architecture style in which large application is split into multiple small applications (aka services). Each service performs a small specific task of the application hence called micro-service. Each microservice can use its own set of technologies (language, db etc). The services communicate with each other using queues or REST API. Read Martin Fowler’s [excellent guide](http://martinfowler.com/microservices/#what) on the topic. Following diagram is from my experience converting [monolith 
application to microservices]({{site.url}}/projects/accidental-microservices/). 

<figure>
    <a href="{{ site.url }}/images/blog/microservices.png"><img src="{{ site.url }}/images/blog/microservices.png"></a>
    <figcaption> A sample microservices based application </figcaption>
</figure>

Since each microservice is small, a few/all of them can be deployed on a single server. There are 2 ways of doing this: Virtual Machines and Containers.

## Virtual Machines
An entire Operating System wrapped in a installable software. You can install your application on a single OS, wrap it in a Virtual Machine and then install it on any server (OS/Linux/Windows) and application will work correctly. VMs are resource heavy and large (since it is an entire OS), thus it is slow and you can only use limited number of VMs on single server. 

## Container Runtime
Instead of full-fledged Operating Systems we can use set of core libraries that most applications use to interact with the server. Container engine is a thin layer that acts as an intermediary between application and OS/Other-applications. 

<figure>
    <a href="{{ site.url }}/images/blog/microservices/container-vs-vm-2.jpg"><img src="{{ site.url }}/images/blog/microservices/container-vs-vm-2.jpg"></a>
    <figcaption> Virtual Machines (left) vs Containers (right) </figcaption>
</figure>

## Containers 
Containers are thin wrappers surrounding the microservice/application. Like VMs if your application works well within the container, you can be assured it will work on any server (OS/Linux/Windows). If application is wrapped in a container, it can be easily replicated or migrated to another server. (example: replicate front-end container in response to increase in requests). Also, it ensures, containers restrict application’s abilities on the server. (Example: application can access only restricted set of users, file system, ports, CPU, memory etc). 

## Container Formats
These containers need to have specific format so that the runtime can start and run them properly. There are mainly 2 competing formats 

- [Docker](https://www.docker.com/what-docker)
- [Rkt by CoreOS](https://coreos.com/blog/rocket/)
- [OCI](https://www.opencontainers.org/about) Attempt to standardize container format

### Side note: 

Industry has [not](https://sreeninet.wordpress.com/2016/06/11/container-standards/) [yet](https://coreos.com/blog/making-sense-of-standards/) [standardized](https://www.linkedin.com/pulse/forking-docker-daniel-riek) the format for containers. Thus, all container orchestration frameworks advertise their compatibility with different containers. For example: [Kubernetes](http://blog.kubernetes.io/2016/07/rktnetes-brings-rkt-container-engine-to-Kubernetes.html) is capable of running both Docker and Rocket containers. 

Docker and CoreOS are 2 companies which not only have their container formats (Docker and Rkt) but also entire ecosystem to 
 manage them (some of which are listed below). 

## Container Registry

Once a container is created for the application, it can be stored on a server called container registry. This helps in reusing the container image, to create new copies/instances. 

Registries are of two types

- Public - Which hosts basic software images like Tomcat, Nginx, MySQL etc.
- Private - Which host your custom application containers. 

Typically, first a basic image is downloaded from [Docker’s Open Repository](https://hub.docker.com). Then the image is customized as per the application’s needs, and then saved to any of the private registries mentioned below.

- [Docker Registry](https://docs.docker.com/registry/)
- [CoreOS Registry](https://coreos.com/products/enterprise-registry/docs)
- [AWS Registry](https://aws.amazon.com/ecr/)
- [Google Container Registry](https://cloud.google.com/container-registry/)

<figure>
    <a href="{{ site.url }}/images/blog/microservices/docker-registry.png"><img src="{{ site.url }}/images/blog/microservices/docker-registry.png"></a>
    <figcaption>Docker's Public Container Registry</figcaption>
</figure>

## Orchestration

Once all microservices/applications are wrapped in containers, there needs to be a way to automate their deployments, rollbacks, restart, healing etc. Also, there is a need to monitor services, discover new services, authenticate them, scale/replicate etc. 

All these requirements call for an orchestration framework. 

Few of such frameworks are listed below:

- [Kubernetes](https://deis.com/blog/2016/kubernetes-illustrated-guide/) - Most popular one. 
- [Docker Swarm](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/) - Docker’s own embedded orchestration.
- [Mesos](mesos.apache.org/) - Highly popular cluster manager, used to manage servers. Is used with Hadoop, Kafka and Spark. Recently gained container (Docker/Rkt/OCI) [support](https://blogs.apache.org/foundation/entry/the_apache_software_foundation_announces97).
- [Amazon ECS](https://aws.amazon.com/ecs/) - AWS’s own cloud based orchestration.
- [Google Container Engine](https://cloud.google.com/container-engine/) - Kubernetes on Google Compute Engine.

<figure>
    <a href="{{ site.url }}/images/blog/microservices/kubernetes.png"><img src="{{ site.url }}/images/blog/microservices/kubernetes.png"></a>
</figure>

## Service Discovery 

As part of the orchestration, each service needs to register itself, so that other services and orchestrator can discover and work with it. For this functionality, a highly available, strongly consistent system is required. Few of softwares which satisfy these requirements include [ETCD](https://coreos.com/etcd/), [Consul](https://www.consul.io/) and  [Zookeeper](https://zookeeper.apache.org/). These projects are used by frameworks internally, and we need not have complete understanding of it. I’d recommend this [Google paper](https://research.google.com/archive/chubby.html) if interested in knowing how these systems are implemented. 

## Conclusion

The ecosystem surrounding microservices and containers is vibrant and fast-paced. Since, it is still going through
the churn and yet to mature, it can be difficult to keep track of all projects. I guess, we will soon see
consolidation of these technologies. Trends are beginning to show Docker leading in containers while Kubernetes
in container management. Fascinating times. 


##  Left over buzzwords

- [Hadoop](https://www.youtube.com/watch?v=d2xeNpfzsYI): System used to store and process large amounts of data. 
- [Kafka](http://kafka.apache.org/): Distributed messaging system.  
- [Spark](http://spark.apache.org/): Big data processing engine. Similar to (& allegedly much better than) Hadoop
- [Redis](http://redis.io): Distributed key-Value store.

If you want to learn about some of the modern databases, check out this [article]({{site.url}}/blog/choosing-databases/).