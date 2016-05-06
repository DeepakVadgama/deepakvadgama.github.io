---
layout: post
title: Database modeling mishaps
category: blog
comments: true
excerpt: How database modelling decisions can hamper or catalyze development and maintenance 
tags: database, domain
---

I have been working on an [ERP software]({{ site.url }}/projects/refactoring-to-sanity/) which was created by 3 developers fresh
out of college. Suffice to say, code base I got was inefficient. Though code in itself can be refactored, correcting the 
 database structure post production is quite difficult. 
 
### Domain

Balaji Wires and Cables company creates wires using simple process flow of 4 steps - make, check-quality, store, deliver

- Stock In = Process of stocking in the item after making it is called 'Stock In'
- Store Register = Once stocked-in, the item is said to be in 'Store Register'
- Quality Check = While in store, cable is tested for quality
- Stock Out = Once quality is tested, the item can be 'Stocked out' and is ready for delivery

<figure>
    <a href="{{ site.url }}/images/blog/balaji/balaji-flow.png"><img src="{{ site.url }}/images/blog/balaji/balaji-flow.png"></a>
</figure>

### Database tables

Notice that during the flow, a *single* item goes through these 4 *states*. Thus it would make sense to have single database table
representing that item. This is called object oriented modelling, where-in all real life object types are distinct tables in DB. 
This way of modelling DB structure works for most of the cases.

Instead, currently, each one of the states has a corresponding table!

<figure>
    <a href="{{ site.url }}/images/blog/balaji/database-modeling.png"><img src="{{ site.url }}/images/blog/balaji/database-modeling.png"></a>
</figure>

#### Problems

- Each of these tables cover roughly 90% of the same fields. So each item is represented thrice in DB.
- To avoid this, the previous team retained only Stock-IN table records, and deleted records from other tables as flow proceeds.
- Though weirdly, in intermediate state, record is present in all 3 tables, so if weight is updated for item, it needs to be synchronized across tables. 
- The Software allows reverting the item to previous state. This means constantly deleting and inserted records in 3 tables.  
- Each of these 3 tables have their own primary key, which are not synchronized. A functionality called 'label' which prints physical sticky labels, uses these keys to uniquely identify the item.  

> Mistakes made in earlier stages especially at a structural level are hard to rectify, even more so over time as the codebase keeps growing, inculcating technical debt.
 
 This whole ordeal could have been avoided, using single table with a column to represent the current state of the item. 
 
 Effort wise, merging these 3 tables into a single one, means changing more than 20 files (cumulative thousands of lines of code) and retesting of 
 all the corresponding functions. With ETA of 2-3 weeks for the whole change, client is more inclined to add features to the software, than risk changing the underlying structure.
  
  Irony is, while we wait for rectifying these structural issues, codebase depending on this precise structure keeps growing, which adds to technical debt. 
  Which means, 6 months from now, rectifying this will take 6 weeks, and more risk of impacting existing functionalities.     
  
So, please model your database structure appropriately. For ERP softwares (especially at small/medium scale), modeling as per real world objects will always work.

> When in doubt, look into the real world