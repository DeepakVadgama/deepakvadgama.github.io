---
layout: post
title: Design Air Traffic Controller
category: blog
comments: true
published: false
excerpt: Walk-through for a system design interview question  
tags:
  - interview
  - design
---

Model interactions between components.

## requirements

flights,
 atc
 runway
parking lot with x capacity, 
n gates

encapsulate scheduling in flight-requests-queue 

Runway.isBusy

- async (event based) communication between flight and atc

system for gates
system for parking lot
wrapper system for (occupancy?)
