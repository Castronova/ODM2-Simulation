## Description
This document outlines the desirable functionality of the Simulations extension through common SQL queries.  The goal of the document is to identify common SQL queries that will be used to interact with the database to ensure that they will be made possible.

---



### Find the ActionID of a Simulation
The actor wants to identify the Action associated with a simulation instance.

```sql
SELECT a.ActionID FROM Actions a
JOIN Simulations s ON s.ActionID = a.ActionID
WHERE s.SimulationName = "Logan SWMM Model" AND a.BeginDateTime = "2014-04-22 12:55:00"
```

### Find all Decendents of a Simulation
The actor wants to identify all simulations that have been produced as children of a specific model.

Given:

| Table | Column | Value |
|:---|:---|:---|
|Simulations | SimulationName | Logan SWMM Model |

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



### Find all Grandchildren of Parent Simulation
The actor selects all simulations that are siblings of the same parent simulation.
Given:

| Table | Column | Value |
|:---|:---|:---|
|Simulations | SimulationName | Logan SWMM Model |

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

### Select Geometry of Simulation Result
The actor wants to determine all geometries of a simulation output Variable.

### Select Simulation Inputs
The actor wants to select all input Variables and Units for a specific simulation.

### Select Simulation Outputs
The actor wants to find all simulation output variables and units that were produced by a simulation.

### Update Input DataSet
The actor wants to associate a Results measurement as an input to a simulation.

### Select Simulation Parameters
The actor wants to select all input parameters and their values for a specific simulation.

### Find all Simulations Created by an Organization
The actor wants to discover all simulations published by an organization by name

### Find all Simulations that have Variable Output
The actor wants to find all simulations that produce an output of type Variable and Unit.