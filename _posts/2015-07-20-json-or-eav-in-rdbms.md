---
layout: post
title: JSON or EAV in RDBMS
---
NoSQL Document DB like MongoDB provides flexibility without built in schema, which means you can add any field, array, hash into a JSON document without changing DB schema. Can we get the same flexibility in RDBMS like MySQL Database?

I met a real case recently that I want a common set of database model to be applied to different services. For sure, there're some common fields, tables can used by all the services. However, there're still some items left, which cannot be aligned. Also, there's no guarantee these fields won't be changed later. It means we need the same flexibility, somehow "no schema" like MongoDB.

Here is some solution:

1. JSON field in RDBMS (MySQL)
2. EAV style

Let's take a look at them one by one.

## 1. JSON field in RDBMS (MySQL) ##

In short, there is no way to work with JSON in MySQL. You can definitely store JSON field in MySQL. However, you cannot utilize search, order functionalities via SQL that MySQL provides. It means you have to load everything into your memory and operate them in code.

There is a pre-release in MySQL 5.7 for [JSON functions](http://labs.mysql.com/). But they are really in early stage. No suggest to use it in real projects.

## 2. EAV style ##

EAV means Entity-Attribute-Value (also sometimes 'name-value pairs'). Here is an example

#### Nomral model: ####

    ID   Ht_cm   wt_kg   Age_yr  ... 
    1    190     82      43      ...
    2    170     60      22      ...
    3    205     90      51      ...

#### EAV: ####

    ID  Metric   Value
     1  Ht_cm    190
     1  Wt_kg    82
     1  Age_yr   43
     1  ...
     2  Ht_cm    170
     2  Wt_kg    60
     2  Age_yr   22
     2  ...
     3  Ht_cm    205
     3  Wt_kg    90
     3  Age_yr   51
     3  ...

There are also plenty of reasons you shouldn't use EAV:

1. No way to define data types.
2. No easy way for contraints.
3. No unique index.
4. Hard to query if need select with multiple EAV fields.

#### For query, compare: ####

    SELECT cID.ID AS [ID], cH.Value AS [Height], cW.Value AS [Weight], cA.Value AS [Age]
    FROM (SELECT DISTINCT ID FROM Client) cID 
    LEFT OUTER JOIN 
        Client cW ON cID.ID = cW.ID AND cW.Metric = "Wt_kg" 
    LEFT OUTER JOIN 
        Client cH ON cID.ID = cH.ID AND cW.Metric = "Ht_cm" 
    LEFT OUTER JOIN 
        Client cA ON cID.ID = cA.ID AND cW.Metric = "Age_yr"


#### To: ####

    SELECT c.ID, c.Ht_cm, c.Wt_kg, c.Age_yr
    FROM Client c

However, as most of these limitations except #4 (hard to query) also exist even in MongoDB. They are just another side of the coin, no schema. And I can live with that at least.

## Summary ##
No-schema flexibility is not natural in RDBMS like MySQL. You need to consider carefully while choosing EAV or using NoSQL DB instead.
