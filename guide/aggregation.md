---
title: Aggregation
description: Use Aerospike Stream UDFs to aggregate query results in a distributed fashion. 
assets: /docs/guide/assets
---

<div style="float: right" >
![]({{book.assets}}/icon-aggregations.png)
</div>

Use Aerospike Stream UDFs to aggregate query results in a distributed fashion. The Aerospike aggregation framework allows fast and flexible query operations. This programmatic framework is similar to a MapReduce system, in that an initial map function runs over a collection and emits results in a highly parallel fashion. Results traverse a pipeline of either subsequent map steps or reduction steps and aggregation steps.

Unlike Hadoop or other frameworks using Java, Aerospike aggregation is implemented using Lua. Each client sends an aggregation request to all servers in the cluster, which independently count the results and return individual results to the requesting client.

The Aerospike aggregation framework differs from other systems because Aerospike recommendeds running aggregation against an index; this is essentially a `where` clause. Filtering against an index maintains high performance. Aerospike supports aggregation for tables and the entire namespace. The client then runs a final reduce phase, also in Lua, to sum results.

## Use Cases

- Implementing aggregate functions such as SUM, COUNT, MIN, MAX as user defined
  stream UDFs.

- Real-time dashboarding. 

  Using secondary indexes on bin with an update time, aggregations quickly gather statistics on recently changed records. Aerospike aggregation touches fewer records compared to standard MapReduce systems, which act on the entire unindexed dataset.

## Performing Aggregation

<div style="float: right" >
![]({{book.assets}}/query_stream_mapaggregatereduce.png)
</div>

To implement aggregation in your application: 

1. Create a [query application](/docs/guide/query.html#create-a-query-application).
 1. Create indexes on a bin.
 1. Insert a record in an indexed bin.
1. Create an [aggregation module](/docs/udf/developing_stream_udfs.html) in Lua. 
1. Set the aggregation module path. 
1. Register the module with the Aerospike cluster.
1. Construct an aggregate query with a predicate (`where` clause).

### Aerospike Clients

Write queries using these language-specific examples in the Aerospike client libraries:

* [Java](/docs/client/java/usage/aggregate)
* [C# .NET](/docs/client/csharp/usage/query/aggregate.html)
* [Node.js](/docs/client/nodejs/usage/query/aggregate.html)
* [PHP](/docs/client/php/usage/udf/aggregate.html)
* [Python](/docs/client/python/usage/udf/aggregate.html)
* [C](/docs/client/c/usage/query/aggregate.html)

### AQL

This aggregation process uses the [aql](/docs/tools/aql) command line tool.

**Create an Index**

This example aql script creates a string index on the *user_profile* namespace, set *west*, and bin *location*.

```bash
aql> CREATE INDEX ix2 ON user_profile.west (location) string
OK, 1 index added.
```
{{#note}} 
Before running this query, ensure that the *aerospike.conf* file contains the *user_profile* namespace. If *user_profile* is not present, use the following script to add it. 
{{/note}}

```bash
namespace user_profile {
    replication-factor 2
    storage-engine memory
}
```

These examples use the in-memory storage engine. Queries can also run in on-disk namespaces.

**Insert Data**

{{#note}}
Although aql is not intended to be used by applications, you can use a Java code snippet to insert data (see the [Java Client&mdash;Synchronous](/docs/client/java/usage/kvs/write.html) database write example).
{{/note}}

This example aql script inserts records to prepare for an aggregation. 

```bash
aql> INSERT INTO user_profile.west (PK,location,last_activity) VALUES ('cookie_100','MA',342)
OK, 1 record affected.
aql> INSERT INTO user_profile.west (PK,location,last_activity) VALUES ('cookie_101','AZ',345)
OK, 1 record affected.
aql> INSERT INTO user_profile.west (PK,location,last_activity) VALUES ('cookie_102','CA',345)
OK, 1 record affected.
aql> INSERT INTO user_profile.west (PK,location,last_activity) VALUES ('cookie_103','AL',340)
OK, 1 record affected.
aql> INSERT INTO user_profile.west (PK,location,last_activity) VALUES ('cookie_104','TX',347)
OK, 1 record affected.
aql> INSERT INTO user_profile.west (PK,location,last_activity) VALUES ('cookie_105','MA',323)
OK, 1 record affected.
```

**Create an Aggregation Module** 

This Lua code example counts the number of users in a location, using map-reduce instead of an aggregation step. The `aggregate` function is more efficient, but map-reduce is more flexible.

```lua 
file: aggregate.lua

function count(s)
    function mapper(rec)
            return 1
    end
    local function reducer(v1, v2)
        return v1 + v2
    end
    return s : map(mapper) : reduce(reducer)
end
```

Where the stream operations are:

- `mapper`&mdash;Returns an integer (1) for each profile record.
- `reducer`&mdash;Adds all 1s for a total count. 

**Set the Aggregation Module Path**

You must set the aggregation module path on the client side because the final reduce phases run on the client side once a response is returns from all cluster nodes. Use aql to set the relative path to the Lua aggregation module.

This example aql script sets the path. Note that *aggregate.lua* is in */home/user/lua_code* and aql started in */home/user*. 

```bash
aql> set LUA_USERPATH  'lua_code'
```

**Register the Module with the Cluster** 

This example aql script registers the Lua aggregation module with the Aerospike cluster. 

```bash
aql> register module 'lua_code/aggregate.lua'
OK, 1 module added.
```

{{#note}}
Once the module registers with one cluster node, Aerospike replicates it on all cluster nodes.
{{/note}}

**Construct the Aggregate Query**

This example aql script constructs the query.

```bash
aql> AGGREGATE aggregate.count() ON user_profile.west WHERE location='MA'
+-------+
| count |
+-------+
| 2     |
+-------+
1 row in set (0.007 secs)
```

## References

- [aql&mdash;Aggregating on Query or Scan Results](/docs/tools/aql/querying_records.html#aggregating-on-query-or-scan-results)
