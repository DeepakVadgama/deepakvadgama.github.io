---
layout: post
title: Choosing Databases
category: blog
comments: true
excerpt: Brief taxonomy of modern databases.
tags: database
---

<figure>
 <a href="/images/blog/database.jpg"><img src="/images/blog/database.jpg"></a>
</figure>
 
In last 5-10 years there has been huge wave of startups attempting to re-invent databases. This re-invention is justified by advent of
   
+ **Scale** - Proliferation of mobile. Billions of potential users. Hockey stick user growth. These require radical rethink of architectures.  
+ **Mobile (Web+Apps)** - Varied platforms, intermittent connectivity and lean startup principle of constant changes, require a standardised + flexible format (JSON), simplification of synchronization of user data and offline capabilities built into the client.  
+ **Analytics** - For converting scale into value, massive data needs to be analyzed, which calls for structuring data differently (eg: Graph DB for Social Networks, BigTable for storing billions of Web pages).  
+ **Computing** - Constantly reducing storage (HDD/SSD/RAM) prices and improved seek performance (SSD) have enabled new DB engine architectures (moving away from BTables and slow seeking disk-blocks).    
+ **Cloud** - Cloud computing has enabled developers to focus on data itself; offloading major pain points of complicated aspects like sharding, distributed coordination, consistency etc.  

## Developers / Startups 
For consultants/SaaS startups, most important features of DB are:  

- For server - SQL. Reliability. Performance.
- For Web/Mobile - JSON storage. Dashboard. Offline and Sync.

Thus, I will not mention many important but not-required-by-everyone features. 

## Classic Relational DB - Schema-based - SQL

+ [MS SQL](http://www.microsoft.com/SQLServer‎)- By Microsoft. Licensed.
+ [OracleDB](https://www.oracle.com/database/) - By Oracle. Licensed. 
+ [MySQL](http://dev.mysql.com/downloads/) - Best, free, relational DB. Though now owned by Oracle. 
+ [PostgreSQL](http://www.postgresql.org/) - Another hugely popular, free, relational DB.
+ [MariaDB](https://mariadb.org/) - Fork of MySQL after it was bought out by Oracle. Led by founder of MySQL.
+ [WebScaleSQL](http://webscalesql.org) - Fork of MySQL for large scale deployments. Joint effort of Facebook, Google, Twitter, Alibaba. 
+ [CockroachDB](https://www.cockroachlabs.com) - Created by ex-employees of Google, Facebook & Twitter. Scalable, distributed SQL with strong focus on availability.


## Schemaless DB - NoSQL
Divided into 4 types - Document stores, Columnar DB, Graph DB and Key-value stores

### Document stores - aka Store JSON objects.  
All these have API, Dashboard, Indexing and Querying facilities. 

+ [MongoDB](https://www.mongodb.com/) - Document store DB pioneers. Loosing steam lately?
+ [CouchDB](http://couchdb.apache.org) - MongoDB cousin. Apache project. Offline capabilities. 
+ [PouchDB](http://pouchdb.com) - CouchDB's cousin in browser. Works across all browsers. Syncs well with CouchDB on server. 
+ [RethinkDB](https://www.rethinkdb.com/) - Push JSON from DB into UI/Server.
+ [Firebase](https://firebase.com) - Same as RethinkDB, but hosted. More capabilities in cloud section below. 

 
### Columnar DB - Wide Column DB
+ [Cassandra](http://cassandra.apache.org/) - To store loose-schema based data in form of rows/columns/column-families. Google’s BigTable cousin. For large scale data. You probably don’t need this. Lets turn back.

### Graph DB
+ [Neo4J](http://neo4j.com) - For Graph based databases like if you want to model social connections of Facebook or LinkedIn. Again, we don’t need this. Lets turn back.
 
### Key-Value stores

+ [MemcacheD](https://memcached.org/) - In-memory (non-persistent) key-value store.  
+ [Redis](http://redis.io) - Persistent Key-Value distributed storage. Not DB per se. Typically used for caching/task-queues. This topic is going tangential. Lets head back to DB. 


## Cloud - Hosted Solutions

+ [Firebase](https://firebase.com) - NoSQL. With offline, sync, dashboard, security-rules. 
+ [Google Cloud SQL](https://cloud.google.com/sql/), [Amazon RDS](https:/aws.amazon.com/rds‎), [Microsoft Azure SQL](https://azure.microsoft.com/en-in/services/sql-database/) - SQL in the cloud.
+ [Amazon DynamoDB](https://aws.amazon.com/dynamodb/), [Microsoft Document Store](https://azure.microsoft.com/en-in/services/documentdb/), [Google Datastore](https://cloud.google.com/datastore/), [Google BigTable](https://cloud.google.com/bigtable/) - NoSQL in the cloud.

## Cloud - Object store

+ [Amazon S3](https://aws.amazon.com/s3/) - Key-Object store. Object = images, videos, files etc. Group into buckets. Security rules. API.
+ [Google storage](https://cloud.google.com/storage) -  Same as above.
 
## Cloud - Archival Storage aka Cold Storage

+ [Google Nearline](https://cloud.google.com/storage-nearline/) - Archival storage. Store + Retrieve price 1 cent/GB. 3 secs retrieval time.  
+ [Amazon Glacier](https://aws.amazon.com/glacier/pricing/) - Archival storage. Cheap to store, expensive to retrieve. 
 
## Conclusion:

+ **Cloud computing**
Thanks to cloud computing, developers need not worry about complicated aspects of availability, sharding, load-balancing, replication, backups, performance, cost etc. All these are taken care of by capable cloud providers. 

+ **Mobile**
Most NoSQL (document store) database solutions are (fast) converging onto similar set of capabilities. Dashboard, Offline, Push, Sync, Security Rules, REST API, Cloud hosted etc. 

Databases have come quite far and its looking bright for developers. I personally use MySQL currently (will move to Google Cloud SQL), and plan to give Firebase a try for specific use cases.
