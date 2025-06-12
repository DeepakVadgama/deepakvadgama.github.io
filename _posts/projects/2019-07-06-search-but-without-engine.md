---
layout: post
title: '[2019] Search, but without an engine'
category: projects
comments: true
published: true
excerpt: Building an API aggregator 
---

I have been working on a federated search model since 2 years until March 2019, where for every query, we make API calls to all search providers,
aggregate the results and send back the response. Though, it sounds very limited, we were able to add layers of functionality on top to make it better for our users. 

### The Product

- Initially our team owned both the UI (Web portal) and backend for search. 
- Eventually we exposed search as a service (API) 
- API has been adopted across 4 channels (Web, Mobile and 2 Desktop applications). 

### Basic flow

When Search API is called, user is authenticated. For each provider entitlements are checked if user is allowed to call the providers. 
All entitled providers' search APIs are called in parallel, with a timeout of 2 seconds. Returned results are converted
to JSON format, aggregated and returned to the user. 

### Slow provider problem

- Normal execution - Create 7 Runnable tasks, a countdown latch with timeout of 2 seconds and submit all tasks. Aggregate and return results arrived within 2 seconds. Average latency across providers was 300ms, so we hardly breached 2 second barrier. 
- Slow provider - Sometimes a provider responds slow for a period of time. One of the 7 tasks takes more than 2 seconds to complete. Individual search query is not affected as countdown latch breaks and partial results are returned. 
- But, if search rate is high, tasks for slow provider eventually occupy the queue and thus, all threads. 
- Such tasks are short-ciruited if result for that search is already returned. 
- This is very similar to circuit breaker pattern used for fault-tolerant microservices.
- One API client which wanted the provider result irrespective of time-taken used API parameter of timeout (increasing it to 10). This had potential to impact our
servlet thread-pool but after monitoring for a few days decided against putting extra precautions. 

### Search API

- License key for onboarded users (plan was to use it for rate-limiting & metrics)
- API allowed specifying providers, max result count & custom timeouts
- Slow providers not included by default. Could be explicitly added to the request.
- JSON response contained custom schema structure for each provider. 

### Entitlements

- Authentication was done by an in-house service (available as Apache plugin & Cloud Foundry services).
- Some providers (legacy systems) used their own userids. These ids were fetched via an API and cached in Solr. 
- Authorization of providers for each user was done using entitlements service and cached for 4 hours. 

### Configuration & Feature flags

- All the features are kept behind a feature flag
- It helped immensely during a production bug which was causing too many calls to a legacy service
- Disabled the feature by flipping the flag & avoided emergency bug fix deploy.
- New config is fetched every 10 minutes. 
- Protected (admin only) API to force flush the cache & reload. 

### Recent Queries

- API to return 5 recent search queries done by the user 
- Stored in Solr, and least recent removed simultaneously.
- Solr, technically not the right technology for this but it was available to us as a managed service 

### Recent Navigations

- API to return 5 recent search results clicked (navigated to) by the user 
- JSON for the result stored on every result click in Solr
- If more than 5 results present for the user, least recent removed simultaneously.
- Solr, technically not the right technology for this but it was available to us as a managed service 
- Once a day, all results checked for entitlements.
- Entitlement check tasks (users entitlements already available in cache)
- For one provider (w/ fine-grained entitlement), API call was made to check entitlement (detailed below).

### Trending Navigations 

- API to expose top 5 trending results (entitled to be seen by the user)
- Co-relational to news/market events
- Top 100 trending items retrieved from metrics (for last 7 days)
- When API hit for a user, each result/item checked for entitlement in batches of 5 (ExecutorService)
- Trade off between eager load (too much memory use) vs lazy load (first time delay in response as results are not cached)
- 5 results cached in Solr against the userid
- Flushed every day, and new top 100 retrieved

### Auto-suggest / Synonyms

- Ability to suggest search queries to the user.
- Used Solr's suggest API.
- Populated with previous searches by the users (extracted from metrics).
- Also, added reference data (Analyst, Companies, Tickers, Currencies, Countries etc.) to it.

### Fine grained entitlements

- One data provider maintained its own set of fine-grained entitlements (not available to us). 
- This limited the trending and recent navigation functionality, where user entitlements had to be checked at regular intervals. 
- Provider API endpoint was hit for entitlement checks, where the response was matched with the cached result. If it matched, that meant user still had the entitlement for that item. 

### Cloud Foundry

2 separate replicas of architecture (with common Solr/DB)

*Old*
- Apache (w/ plugin for authentication)
- Tomcat (4 instances) on VSI boxes
- Used only for web portal

*New*
- 3 new data sensitive providers onboarded. Had to replicate architecture in more secure zone. 
- Chose in-house Cloud Foundry. Our application was already stateless and adhered to 12-factor, thus required minimal changes to the code-base.
- 9 Cloud Foundry instances across 3 regions (9 AZs)
- Global Load Balancer with active health checks.
- Used for Search as a service API
  
### Rest of the stack

- Maven multi-module to ensure separation of code connecting to data sensitive providers. Each build creates 2 artifacts one for each architecture copy.
- Jenkins to build and deploy artifacts (one using in-house model other using cloud foundry push command)
- Splunk to view the logs
- Metrics service to store the events
- Retrofit for HTTP calls
- Solr for caching


