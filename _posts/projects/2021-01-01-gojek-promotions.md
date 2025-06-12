---
layout: post
title: '[2021] Gojek food delivery campaigns'
category: projects
comments: true
published: true
excerpt: Building Food Delivery Promotions
---

## Background

GoFood is [GoTo](https://en.wikipedia.org/wiki/GoTo_\(Indonesian_company\))'s online food delivery platform, operating in Indonesia. As of 2019, GoFood processed 1.5M+ orders a day connecting users with a network of 1M+ merchants (aka restaurants) and 2M+ driver partners. During COVID-19 (2019-2021), GoFood experienced significant growth across key indicators.

* Monthly Active Users (MAU): 150% (8M \-\> 20M)
* New Merchants: 87% (400K \-\> 750K)
* Food Orders: 72% (1.2M \-\> 2M)
* Gross Merchandise Value (GMV): 183%

Discounts are used as a key mechanism to influence business metrics. These business goals can be diverse, [CLTV](https://en.wikipedia.org/wiki/Customer_lifetime_value) (Customer Lifetime Value), AOV (Average Order Value), overall GTV (Gross Transaction Value), customer stickiness, operational efficiency, CSAT improvement etc. To achieve these personalized discounts are utilized with dimensions like customer restaurant affinity, location aware demand shaping, delivery vs cart discounts, discount visibility etc.

As of this project initiation, the discount mechanisms available in GoFood are menu-item flat-value discounts (created by merchants), delivery discounts (created & funded by Gojek) and vouchers (deprecated).

## Business Goals

Existing discounting mechanism is severely limited in its capabilities. The menu-item level discounts comprise of 75% of discounts. Delivery discounts which are applicable to a service-area and are visible only on checkout page restricting their effectiveness.

The new feature is aimed at improving the following business metrics:

* Increase average order value (AOV) by 70%
* Increase CSAT score from 2.8 to 4+
* Increase customer retention (new / churned users, order frequency)
* Increase Merchant Funded Promotions to $10M per month

## Functional requirements

Business needs ability to create highly personalized discounts based on following factors:

* Applicability \= Service area, a restaurant or franchise
* Duration (granularity of 1 day)
* Discount \= Percentage or a fixed amount
* Targeting \= user segments (include and exclude)
* Minimum order value threshold
* Maximum campaign amount
* Redeemable limit (per user, per campaign)
* Stackable discounts (configurable)

Product capabilities

* Increase the visibility across all touchpoints in the app (Home, Search, Restaurant Profile, Checkout pages)
* Merchants should be able to self create campaigns.
* Campaigns can be Gojek funded, Merchant funded or Co-funded.
* Campaigns can be activated or deactivated instantly (but not paused or resumed).
* Build subscription product on top
* Architecture should be extensible for future features like time-based discounts, promo codes, subscription and payment method specific discounts.

# Design

## Existing architecture

The delivery discounts are stored in ordering system and only shown to the user on checkout screen. The menu-item discounts are time-based (eg: applicable only on Mon-Thu between 5pm to 7pm). Every hour the discount service fetches the discounts which are to be activated and sends them to the content service. Most discounts don't use granular time and only rely on dates which increases the midnight activations & deactivations to \~750K.

This push based system creates high traffic at midnight. To avoid overwhelming the content system with write spikes, the updates are delivered asynchronously via Kafka. Due to high volume, these writes are queued at Kafka and can cause delay in activating / deactivating discounts causing suboptimal merchant experience.

<figure>
 <a href="{{ site.url }}/images/blog/gojek/existing_architecture.png"><img src="{{ site.url }}/images/blog/gojek/existing_architecture.png"></a>
</figure>

## New Architecture

The architecture proposed having discount service as a single source of truth for all campaigns. Instead of a push system, any touchpoint on the app or a service which needs to know would fetch it from discount service.

### Campaign Creation & Updates

Discount service will be a stateless service that will expose API to create & update campaigns.

Creation: During campaign creation flow, if the start date is current date, the campaign will be considered active and Redis cache will be updated immediately. We rely on Redis's single-threaded model and HashSets to address concurrency and duplication.

* Campaigns cache \= Redis string (key=local-date+campaign-id, value=campaign-metadata). TTL \= campaign expiry date \+ 10 minutes.
* Service-area campaigns cache \= Redis string (key=local-date+service-area-id, value=campaign-metadata).
* Merchants cache \= Redis HashSet (name=local-date+merchant-id, value={campaign-ids}). TTL \= midnight (12am).

|  | Update merchant campaigns in-place | Pre-compute next day's campaigns |
| :---- | :---- | :---- |
| Delay in campaign activation / deactivation | Medium risk if scheduled campaigns are high | No risk |
| Storage need | Single copy of merchant→campaign mapping | 2x merchants |
| Complexity | Low. Any campaign event will can be immediately updated in cache | Medium. Need to build a cron-like system to keep precomputing. |

Deactivation: During deactivation, campaign cache metadata will be updated with status as DEACTIVATED.

Scheduled Activation: Everyday at 11pm campaigns which are supposed to be activated or deactivated next day will be pushed to cache by a worker (a cron like system running within the same process). This flow is extensible to run every T+30 minutes for campaigns that have to be activated/deactivated at T+60 minutes with minimal changes.

<figure>
 <a href="{{ site.url }}/images/blog/gojek/crud.png"><img src="{{ site.url }}/images/blog/gojek/crud.png"></a>
</figure>

### Surface discounts across touchpoints

Discount service will expose separate APIs with view-model associated with corresponding page (eg: home page requires service-area discounts, search listing requires a consolidated maximum discount number, restaurant profile page needs listing of all applicable discounts etc.)

<figure>
 <a href="{{ site.url }}/images/blog/gojek/campaign_cache.png"><img src="{{ site.url }}/images/blog/gojek/campaign_cache.png"></a>
</figure>

<figure>
 <a href="{{ site.url }}/images/blog/gojek/touchpoints.png"><img src="{{ site.url }}/images/blog/gojek/touchpoints.png"></a>
</figure>

### Discount redemption

Exhaustion of user limit per campaign and overall campaign budget are tracked asynchronously using Order events.

<figure>
 <a href="{{ site.url }}/images/blog/gojek/budget_limit.png"><img src="{{ site.url }}/images/blog/gojek/budget_limit.png"></a>
</figure>

### Subscriptions

Subscriptions product is built on top of campaigns. Each subscription has an applicable user-segment. One user purchases the subscription, the user-id is added to the segment. Rest of the flow remains the same.

<figure>
 <a href="{{ site.url }}/images/blog/gojek/subscription.png"><img src="{{ site.url }}/images/blog/gojek/subscription.png"></a>
</figure>

### Monitoring / Rollout

Due to high rollout time for Android & iOS apps updates & install rate, the API and UI changes were prioritised and rolled out behind a flag. This helped with Dogfooding the changes.

Infrastructure and operational metrics and alerts were created using Grafana and InfluxDB focussed on availability and latency for all upstream services. The predictable [diurnal pattern](https://en.wikipedia.org/wiki/Diurnal_cycle) of food delivery domain helped provision right-sized infrastructure.

# Outcome

Over the 18 months post building the service following business outcomes were achieved

* Created $150M in Merchant Funded Promotions
* AOV increased by 48% for discounted orders
* CSAT score for discounts increased to 4.1
* Positive feedback from Business for campaign tooling

<figure>
 <a href="{{ site.url }}/images/blog/gojek/product.png"><img src="{{ site.url }}/images/blog/gojek/product.png"></a>
</figure>

# Challenges

* Influence:
  * To move away from push based vs pull based campaigns. HoE wanted only 1 system (Content system) which would need to be highly reliable.
  * Worked across Content, Search, Merchant Platform and Ordering teams to implement changes.
  * Worked across functions (PM, QA, BA) throughout the project duration (6 months).
* Business analysts accidentally created discounts which were too large, or too broad. Added validations (eg: service area absent, discount thresholds etc.)
* Due to asynchronous nature of campaign limit and user-limit thresholds, more discounts were given out. Deactivated campaigns at 95% threshold to address.
* Availability impacted due to user segmentation service availability. Worked with segmentation service to ensure high uptime.

# What next

* Time-based discounts (eg: hourly discounts)
* Payment method based discounts.
* Caching of user-segments to reduce dependency on segmentation service.
* Automated fail-over for Postgres instances.
* Explore feasibility of in-app cache with affinity & sharding
* Payment method based discounts.
* Promo codes for opt-in discounts (instead of opt-out)