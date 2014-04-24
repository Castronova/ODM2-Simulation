## Standard Use Cases for the ODM2 Simulations Extension 
 The goal of the document is to identify and catalog the desired functionality of the simulations database extension and provide SQL queries to verify that this functionality is possible.  These use cases will form the basis of a unit testing framework to ensure that modifications  to the ODM2 database schema do not break the desired functionality of the simulations extension. 

***Note:*** All SQL examples are performed using a PostgreSQL 9.3.1 database.

### Index

* [Find the ActionID of a Simulation](#find-the-actionid-of-a-simulation)
* [Find all Children of a Simulation](#find-all-children-of-a-simulation)
* [Find all Descendants of a Simulation](#find-all-descendants-of-a-simulation)

---



### Find the ActionID of a Simulation
A user wants to identify the Action associated with a particular simulation instance.  This is a procedure that is likely required prior to querying simulation results.  Querying simulation results will be a core operation used by client-side software to couple model inputs and outputs. In the example below, *begindatetime* in not required, however its inclusion helps distinguish between simulations that may have the same *simulationname*.

| DB Table | Column Name| Value |
|:---|:---|:---|
|Simulations | SimulationName | Logan SWMM Model |
|Actions | BeginDateTime | 2014-04-22 12:55:00 |

```sql
SELECT a.ActionID FROM Actions a
JOIN Simulations s ON s.ActionID = a.ActionID
WHERE s.SimulationName = "Logan SWMM Model" AND a.BeginDateTime = "2014-04-22 12:55:00"
```
---
### Find all Children of a Simulation
The user wants to identify all simulations that have been produced as children of a specific model.  This operation provides the ability to quickly find all simulations that are derived from a parent simulation, which will likely be implemented in client-side software.  This task will be particularly useful for comparing multiple model simulations that have been developed for the same study area. 

Given:

| Table | Column | Value |
|:---|:---|:---|
|Simulations | SimulationName | Logan SWMM Model |
|RelatedActions | RelationshipTypeCV | child |

```sql
select s.simulationid, s.simulationname from simulations s
join actions a on a.actionid = s.actionid
join relatedactions ra on ra.actionid = a.actionid 
where ra.relationshiptypecv = 'child' and ra.relatedactionid =
  (
  select a.actionid from actions a
  join simulations s on s.actionid = a.actionid
  where s.simulationname = 'Logan City SWMM Simulation'
  ) ;
```

Click to see an [example](http://sqlfiddle.com/#!15/a86db/1)!


<!-- 
---
### Find all Grandchildren of Parent Simulation
The actor selects all simulations that are grandchildren of the same parent simulation.

Given:

| Table | Column | Value |
|:---|:---|:---|
|Simulations | SimulationName | Logan SWMM Model |
|RelatedActions | RelationshipTypeCV | child |

```sql
select s.simulationid, s.simulationname from simulations s
join actions a on a.actionid = s.actionid
join relatedactions ra on ra.actionid = a.actionid 
where ra.relationshiptypecv = 'child' and ra.relatedactionid =
(
    select a.actionid from actions a
    join relatedactions ra on ra.actionid = a.actionid 
    where ra.relationshiptypecv = 'child' and ra.relatedactionid =
    (
        select a.actionid from actions a
        join simulations s on s.actionid = a.actionid
        where s.simulationname = 'Parent Simulation'
    )
);
```

Click to see an [example](http://sqlfiddle.com/#!15/7d62e/4)!
-->

---

### Find all Descendants of a Simulation
The actor wants to find all simulations that are descendants of the same parent simulation, or all descendants of all root simulations.  To be considered a root simulation, the database record must not implement the RelatedActions table, i.e. it has not related actions.  The primary use of this functionality is to  discover all simulation records in the database and display them hierarchically using client side software.

***Find all descendants for all root simulations***
```sql
with recursive parents(simulationname,actionid,relationshiptypecv,relatedactionid, depth,path)
AS (
    select s.simulationname, a.actionid, ra.relationshiptypecv, ra.relatedactionid, 1::INT as depth, array[a.actionid] as path
    from simulations s
    join actions a on a.actionid = s.actionid
    left join relatedactions ra on ra.actionid = a.actionid -- Use left join to keep nulled fields
    where ra.relatedactionid is null -- All root models will have a null relatedactionid

    union all

    select s.simulationname, r.actionid, r.relationshiptypecv, r.relatedactionid, p.depth + 1 as depth, (p.path || r.actionid)
    from parents as p, simulations as s
    join actions a on a.actionid = s.actionid
    join relatedactions r on r.actionid = a.actionid
    where r.relatedactionid = p.actionid
  )
select * from parents;
```

*Sample Results*

|SIMULATIONNAME | ACTIONID | RELATIONSHIPTYPECV | RELATEDACTIONID | DEPTH |PATH |
|:---|:---|:---|:---|:---|:---|
|Parent Simulation | 1 |(null) | (null) | 1 | 1 |
|Child Simulation 1 |  2 | child |1 |2 | 1,2|
|Grandchild Simulation 1 | 3 |child |2 |3 |1,2,3|
|Grandchild Simulation 2 | 4 | child |2 |3 |1,2,4|


Click to see an [example](http://sqlfiddle.com/#!15/7d62e/111)!

---
### Select Geometry of Simulation Result
The actor wants to determine all geometries of a simulation output Variable.

---
### Select Simulation Inputs
The actor wants to select all input Variables and Units for a specific simulation.

---
### Select Simulation Outputs
The actor wants to find all simulation output variables and units that were produced by a simulation.  The discovery of simulation outputs (i.e. calculations) will be necessary to successfully couple models.  This operation will be performed by client-side software and provide the actor with the ability to link output datasets to simulations input.

Given:

| Table | Column | Value |
|:---|:---|:---|
|Simulations | SimulationName | Logan SWMM Model |

```sql
select v.variablenamecv, u.unitsname, u.unitsabbreviation
from results as r
full join variables v on v.variableid = r.variableid
full join units u on u.unitsid = r.unitsid
where r.featureactionid = 
(
  select a.actionid
  from actions a
  join simulations s on a.actionid = s.actionid
  join featureactions fa on fa.actionid = a.actionid
  where s.simulationname = 'SWMM simulation for Logan UT'
);

```

Click to see an [example](http://sqlfiddle.com/#!15/2cef5/1)!

### Update Input DataSet
The actor wants to associate a Results measurement as an input to a simulation.

### Select Simulation Parameters
The actor wants to select all input parameters and their values for a specific simulation.

### Find all Simulations Created by an Organization
The actor wants to discover all simulations published by an organization by name

### Find all Simulations that have Variable Output
The actor wants to find all simulations that produce an output of type Variable and Unit.