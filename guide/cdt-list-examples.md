---
title: List Examples
description: Examples of modeling with Aerospike lists
---

Aerospike [lists](/docs/guide/cdt-list.html) can be used to implement use cases such as

- Queues
- Unique ordered data
- Time series containers
- Leaderboards
- Membership lists for user profiles

## Modeling Concepts

An Aerospike list can be used to express a many-to-one relationship in a single
record. By comparison in a relational database, such a relationship would span
multiple rows in two tables.

For example, a user and her vehicles would likely be expressed in a relational
 database as a _users_ table and a _vehicles_ table, with a _vehicles.user\_id_
column indexed and declared to be a foreign key to _users_.

In Aerospike a _Users_ set would contain user records. Each user's _vehicles_
could be expressed as a list of map elements, with each map element containing
the information of one vehicle. Users without any vehicles would not have a
_vehicles_ bin at all. In Aerospike, sets have no schema, and records can vary
from one another in their structure. There is no necessity to place-hold
_vehicles_ with something like a `NULL` in a relational database.


## Examples

### Lists as Queues

We have an unordered list `[ 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233 ]`

With the [_aql_](/docs/tools/aql) tool, we will enqueue two new elements at once, using a transaction of two operations.

```bash
aql> OPERATE LIST_APPEND(lqueue, 377), LIST_APPEND(lqueue,477) on test.demo where PK = 'lk1'
+---------+
| lqueue |
+---------+
| 16      |
+---------+
1 row in set (0.001 secs)

OK

aql>  select * from test.demo where PK='lk1'
+-----------------------------------------------------------------------+
| lqueue                                                               |
+-----------------------------------------------------------------------+
| LIST('[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 477]') |
+-----------------------------------------------------------------------+
1 row in set (0.000 secs)
```

We will then dequeue the first two values with one atomic list operation.

```bash
OPERATE LIST_REMOVE_RANGE (lqueue, 0, 2) on test.demo where PK = 'lk1')
+---------+
| lqueue |
+---------+
| 2       |
+---------+
1 row in set (0.000 secs)

OK

aql> select * from test.demo where PK='lk1'
+-----------------------------------------------------------------+
| lqueue                                                         |
+-----------------------------------------------------------------+
| LIST('[1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 477]') |
+-----------------------------------------------------------------+
1 row in set (0.001 secs)
```

We'll use a single transaction to enqueue a record while dequeuing the first record in the queue.

```bash
aql> OPERATE LIST_APPEND(lqueue, 125), LIST_REMOVE_RANGE (lqueue, 0, 1) on test.demo where PK = 'lk1'
+---------+
| lqueue |
+---------+
| 1       |
+---------+
1 row in set (0.000 secs)

OK

aql> select * from test.demo where PK='lk1'
+-------------------------------------------------------------------+
| lqueue                                                           |
+-------------------------------------------------------------------+
| LIST('[2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 477, 125]') |
+-------------------------------------------------------------------+
1 row in set (0.000 secs)

```

### Lists as Containers for Time Series Data

Using the aql tool we'll create a record containing a list of element pairs.

```bash
aql> insert into test.demo (PK, pairs) values ('ll2', 'JSON[[1523474230000, 39.04],[1523474231001, 39.78],[1523474236006, 40.07],[1523474235005, 41.18],[1523474233003, 40.89],[1523474234004, 40.93] ]')
OK, 1 record affected.

aql> select * from test.demo where PK='ll2'
+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| pairs                                                                                                                                                |
+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| LIST('[[1523474230000, 39.04], [1523474231001, 39.78], [1523474236006, 40.07], [1523474235005, 41.18], [1523474233003, 40.89], [1523474234004, 40.93]]') |
+----------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.002 secs)
```

We'll use the Java client to get the pairs within a specified range of timestamp values
```java
Record record = client.operate(new WritePolicy(), key,
                               ListOperation.getByValueRange(binName,
                                      new ListValue(new ArrayList<>(Arrays.asList(1523474231000L, null))),
                                      new ListValue(new ArrayList<>(Arrays.asList(1523474234000L, null))),
                                      ListReturnType.VALUE));

List<?> list = record.getList(binName);

for (Object value : list) {
    System.out.println(value);
    //console.info("Received: " + value);
}
```
Outputs
```
[[1523474231001, 39.78]
[1523474233003, 40.89]
```

