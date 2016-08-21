---
layout: post
title: Making of distributed cache
category: projects
comments: true
published: false
excerpt: How we built distributed cache ground-up to as foundation for our microservices.
---

### Summary
During my tenure at Nomura, [we converted a big monolith based trading  
system into a set of micro-services]({{site.url}}/projects/accidental-microservices).
 It required a base infrastructure to orchestrate communication between all services. 
Instead of using an out-of-the-box solution like [Redis](https://redis.io), we created our own distributed 
cache ground up. The decision was probably an not-built-here-syndrome, 
but for us developers it turned out to be a great learning experience. 

<figure>
    <a href="{{ site.url }}/images/blog/microservices.png"><img src="{{ site.url }}/images/blog/microservices.png"></a>
</figure>


### Building blocks

- Connectivity (TCP based)
- Service Discovery - Similar to zookeeper
- Data serialization format - Kryo
- Caching framework - CQ Engine, Data Listeners
- Remote Procedure Call

### TCP Communication - Grizzly

The base of the communication infrastructure, TCP library was needed to establish and maintain connections
between 2 or more services. We chose [Grizzly](https://grizzly.java.net/) out of many [available for java](https://github.com/Vedenin/useful-java-links#2-networking).
 It performed relatively well except for one issue listed below.

- *Connection pool*: Grizzly [allows]() to tweak the thread pool for connections (useful for throughput). Though we 
made the size configurable, we didn't ever had to change for any service.
- *Listeners*: We added listener with methods for disconnect, re-connect, and heart-beat failure events. The listener
 was exposed to the top most library (caching) using delegation.  
- *Data marshalling and unmarshalling:*
- *Heartbeats:* 
- *Retries:*
- *Backoffs:*
- *Logging:*

### Considerations

1. Heartbeats, Retries, Backoff, Logging at Debug
2. Piggybacking
3. Listeners for connectivity events
4. Indexing cache for quicker retrival
5. Snapshot then realtime updates
6. Ready only after snapshot is complete.. Integrity of data
7. RPC

## Application use-cases 

1. server client cache
2. thin cache streaming 
3. multi client cache
4. lazy cache
5. remoting

Threadpool

Hearbeat
Piggy backing
Multi-client cache  --  datastore for downstream feeds, separate from core system
Stream cache (no storage)
Connection listeners
Connection library - Grizzly - Kryonet alternative - and many others https://github.com/Vedenin/useful-java-links#iii-network-and-integration
Discovery service for startup
Connection timeouts - and logging for that - retry strategies - backoff
CQ Engine (Why instead of hashmap? indexing, sql querying) - IndexedCollection, https://github.com/npgall/cqengine
Snapshot
Kryo - separation of marshalling and unmarshalling
Real time changes - add/delete/update
Snapshot + realtime = default
Dirty cache on disconnect - most caches were of this nature. 
isPresent in cache, get by Key
Value has to extend some interface... so that APIs can be created
Handshake mechanism
Logging - When server disconnects (name of server is known, clients is not known thus log IP)
Alerting (Market disconnected)
Swap collection entirely - atomic change
RMI was built on same connection libraries - 2 types - sync and async (Save clientid + uid of request and compare with response)
IKVM experiment
 