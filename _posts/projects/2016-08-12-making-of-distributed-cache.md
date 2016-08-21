---
layout: post
title: Making of distributed cache
category: projects
comments: true
published: false
excerpt: How we built distributed cache ground-up to as foundation for our microservices.
---

Be clear about what parts you did

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
 