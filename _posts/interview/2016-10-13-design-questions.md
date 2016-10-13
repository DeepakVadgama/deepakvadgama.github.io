---
layout: post
title: Design
category: interview
comments: true
excerpt: Interview questions about design
published: true
tags: 
  - interview
  - notes
---

## Sample 1: Stock rates distribution

Service that will be called by up to 1,000 client 
applications to get end-of-day stock price information (open, close, high, low). Assume that you already have the data, and you can store it in any format you wish. You are responsible for the development, rollout, and ongoing monitoring and maintenance of the feed.
Any technologies you wish, and can distribute the information to the client applications in any mechanism you choose. 

### Questions/Assumptions

- **Volume of data** Is it stored on single machine. Do we need to chunk it before sending to clients. 
- **Type of clients** Web. Backend. Android.
- **Location of clients** External (well-known transport, API, format etc). Internal (binary, if same machine - chronicle etc).

### Considerations

- **API** Ability to request in batches. 
- **Data format** JSON (all clients have parsing libraries, not too verbose as XML) or binary (concise, though not readable, may not be universal) 
- **Transport** Fan out. HTTP 2 or Google Cloud Pub/Sub.
- **Security** Authentication to API. Data encryption (https). Data integrity (hashes).
- **Monitoring** Uptime. Latency. Alerting. (Google Cloud Endpoint)
- **Back-pressure/Retries** Taken care of if data is chunked and clients can request specific chunk


## Sample 2: Sales Rank

 eCommerce company wishes to list the best-selling products, overall and by
category. For example, one product might be the #1 056th best-selling product overall but the #13th
best-selling product under "Sports Equipment" and the #24th best-selling product under "Safety."
Describe how you would design this system. 

### SQL 

- Product table separate 
- Rankings table with columns - category id, product id, ranking
- Category table

When rankings change (example, chromecast jumps from 120 to 2) then need to change rank of all subsequent products. 
In this case, from 2 to 119 add 1 to the rankings... and then change chromecast ranking to 2.

UPDATE RANKINGS  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SET RANKING = RANKING + 1  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;WHERE RANKING >= TARGET_RANKING  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AND RANKING < CURRENT_RANKING  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AND CATEGORY = CHOSEN_CATEGORY 

### Java

- Similar to SQL, have separate collection to maintain rankings and not have it within Product.
- Otherwise to get product rankings for a category, need to iterate over all products.
- Ranking collection can have just the references to the product.

Which collection can store ranked items? TreeMap or PriorityQueue etc but then how will we get Comparator for each category. 
Thus better option is LinkedList.

- **Category** LinkedList for each category
- **Rankings** The (implicit) position in the list is ranking of the product.
- **Updates**: Ranking of each category can be updated independently. 
- **Updates**: Very easy to change ranking of an item. Just update linked-list node reference.
- **Get nth**: This will be O(N) solution but even that can be O(log(N)) if NavigableSkipList is used. 
- **Get nth**: Also, it will be rare to get exact nth item. Rather request will be to get top 10, then 11-20 items (pagination) from the list. 
- **Storage**: Need not cache whole list, lazy load from DB. But will have to cache prefix of the list always, never just middle or end part. And if rankings go to say, #2,152,222 then can have that large list. 
- **Wrap lists**: Have class which stores multiple lists, each list will have start of that list's ranking (so can store any number of chunks from the ranking list of a category). This will also help in doing LRU cache etc. 
 
### Updates
 
 - Updates in real-time or batching is okay?
 - Updates based on popularity or sales or what criteria?
 - 