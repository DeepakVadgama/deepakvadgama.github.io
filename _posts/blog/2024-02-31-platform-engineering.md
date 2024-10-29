---
layout: post
title: 2024 Progress
category: blog
comments: true
published: false
excerpt: Best things and stuff of 2024
---

Resources
[Kubecon Panel talk](https://www.youtube.com/watch?v=yuNTeyDuyhQ)
[KCP talk on Platform Engineering Day Kubecon](https://www.youtube.com/watch?v=az5Rm8Snms4)
[Platform Engineering Maturity Model](https://tag-app-delivery.cncf.io/whitepapers/platform-eng-maturity-model) + [Video](https://www.youtube.com/watch?v=MiYn60VWtJk) + [Presentation](https://www.youtube.com/watch?v=t1DHqnQRuQs)
[CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/)
https://www.youtube.com/watch?v=7op_r9R0fCo
https://newsletter.cote.io/p/platform-engineering-is-just-cicd

- Between IaaS (abstractions on hardware) and Serverless (completely offloading burden of configuration, security etc onto the developers). Platform engineering is something in the middle.

- Success Metrics. Example for X compliance, if you manually do it, how much time you take. X teams x Y hours = number of engineering effort saved. OTher is time to value, how much time it takes to create a new capability, how much time it will take to patch once you find a vulnerability in supply chain (log4j. requires golden paths, ability to control teams' build/configuration, observability), DORA metrics (how fast/frequently teams deploy software), ability to do distributed tracing/observability across service boundaries, how quickly can someone setup an ephemeral test environment

- Categories of platform impact: regulatory (FEDRAMP, Security Auditing, Regionalization), Billing, Quota management, Tenancy management, AuthN/AuthZ,

- It needs product mindset with users are internal users. Users could be application teams, legal teams, compliance teams, databases, even infrastructure teams. It absolutely needs a PM.

- Backstage from Spotify

- Momentum / Burn-out: find exciting greenfield teams who want to experiment, dont boil the ocean accepting requests from every team. Say no, build something that even requires brownfields to change otherwise you will drown in these requests. Needs leadership support for this.

- Enable new capabilities completely impossible before. Example a lens for billing across the company, operational metrics (latency, throughput), resource usage efficiency helping find optimization areas, fault injection testing?,

- Keep giving talks and send newsletters celebrating wins and telling people why what you are doing is important, getting developers onboard.

- User research? Do weekly, 30 min to 1 hour user study where you jump on a zoom with user using the tool to perform a task. Use data and this user research to define the roadmap.

- Drive towards 100% self-serve by building great documentation incrementally, adding contextual help and errors in the tool and checklists / templates to perform tasks

- You can start platform as incredibly small, need not directly jump to IDP / Backstage. Just a set of best practices (configurations, standardized tech stacks etc) then start building tools.

- Interview: product mindset, well-defined success metrics, stakeholder alignment, gradual onboarding, long term grind mindset,
