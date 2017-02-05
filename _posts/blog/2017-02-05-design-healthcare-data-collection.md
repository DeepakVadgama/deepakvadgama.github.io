---
layout: post
title: Design Data Collection System for Health Care
category: blog
comments: true
published: false
excerpt: Walk-through for a system design interview question  
tags:
  - interview
  - design
---

Recent interview question. 

## requirements

- 1.3 crore people
- 3 diseases (diabetes, hypertension, cancer)
- 1 employee = 1000 people
- scheduling regular checkups

## data metrics

Calculate amount of data required.
If needed, claim that more raw data is better in the long run. 

## 3 splits - data collection, storage, analytics

### data collection
- user security
- storing enrollment data (in local DB, primary key?)
- adhaar integration? (not useful and brings more complexities)
- what if employee changes, or device goes down and is replaced

### sending data 
- network security
- network flakiness
- posting data to server (REST?)
- appropriate batch size for sending
- handling response (partial, no record created, failed to parse)
- 

### storage
- load balancing
- proxy
- scaling? 
- kind of server needed (cluster or two?)

### analytics & scheduling
- different reporting database?
- scheduling for medicines or follow ups