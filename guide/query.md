---
title: Query
description: Use Aerospike fast, high-concurrency queries to perform value-based searches on secondary indexes. 
assets: /docs/guide/assets
---
<div style="float: right" >
![]({{book.assets}}/icon-query-2.png)
</div>

Use Aerospike's fast, high-concurrency queries to perform value-based searches using secondary indexes. Secondary Index based queries can return a set of records very quickly, compared to the alternative of doing a table scan. Please see [architecture](/docs/architecture/secondary-index.html) section for more detail on memory cost and run time cost.

With a secondary index, the following query can be made - 
 - **Equal** query against string or numeric indexes
 - **Range** query against numeric indexes
 - [**Point-In-Region or Region-Contain-Point**](/docs/guide/geospatial.html#geospatial-index) query against geo indexes

The results of a secondary index query can optionally be post-processed by Aerospike Predicate Filtering or Aerospike UDFs (user-defined functions) before returning to the client. 

Using Aaerospike Predicate Filtering (Aerospike version 3.12 and above), you can:
- filter out records based on record meta data, such as last-update-time or storage-size.
- filter out records based on bin data, such as integer greater/less or regexp on string bins.

Please see [Predicate Filtering](/docs/guide/predicate.html) for additional details.

Using Aerospike UDFs, you can:

- use map/reduce functions to perform [aggregation](/docs/guide/aggregation.html)
- use [Record UDFs](/docs/guide/udf.html) to touch records
- use [geospatial](/docs/guide/geospatial.html) operators to return Geo location record information

You can fine tune the Aerospike query system based on:

- Query cardinality&mdash;the average number of records returned
- Throughput and latency
- Query load vs. read/write load&mdash;the frequency that the secondary index updates
- SSD parallelism&mdash;the [IOPS](https://en.wikipedia.org/wiki/IOPS) required to support query throughput

See the [Operations Guide](/docs/operations/manage/queries/index.html) for details.

## Secondary Index - Key Data type

Secondary index are on bin values (RDBMS _columns_), similar to a `WHERE` clause in SQL. Secondary indexes keys can only be created on the following primitive data types - 

- [Integer](/docs/guide/data-types.html#integer) data types
- [String](/docs/guide/data-types.html#string) data types
- [Geospatial](/docs/guide/geospatial.html#geospatial-data) data types

In Aerospike, unique secondary indexes are created on a combination of namespace, set name, bin name, key data type, and key source. 

## Secondary Index - Key Source type

There are four index key sources supported in Aerospike. 

- [Basic](#create-a-query-application) wherein key is the bin value
- [List](/docs/guide/cdt-list.html#list-index) wherein key is elements in list
- [Mapkeys](/docs/guide/cdt-map.html#mapkeys-index) wherein key is keys in the map
- [Mapvalues](/docs/guide/cdt-map.html#mapvalues-index) wherein key is value in the map

## Use Cases 

Common queries are:

- top scoreboards, top performers, top earners, and so on
- the most-recent search/query/purchase/activity information from the website front end
- data sets (for example, people with the same employer or who belong to my club)
- multiple IDs such as Facebook name, Twitter handle, and so on
- points of interest near my location

## Limitations
- Aerospike supports up to 256 secondary indexes per namespace
- Fast restart is not supported. On daemon restart, secondary indexes are rebuilt based on record data
- Aerospike is tuned for queries using high selectivity secondary indexes
- For string data-type, only string size <= 2048 can be indexed
- `RANGE` result sets are *inclusive* (that is, both specified values are included in the results)
- If no `set-name` is specified during index creation, then the index will only include records without a set name, but not all sets in the namespace.

## Create a Query Application

To create a query application:

1. [Install](/docs/operations/install/index.html) and configure the [Aerospike server](/download).
1. Create indexes on a bin.
1. Insert records within an indexed bin.
1. Construct a predicate (a `WHERE` clause), make the query request, and process returned records.


### Creating an Index

{{#note}}
Index creation require prior capacity planning, and is typically done by system administrators. An index consumes RAM for every index entry. Background index creation can take a substantial amount of resources. It is important to carefully plan and schedule index creation on production systems. Please see [Secondary Index Memory Requirement](/docs/operations/plan/capacity#memory-required).
{{/note}}

Use the [AQL](/docs/tools/aql/index.html) tool to create and manage indexes in an Aerospike cluster. The aql tool provides an SQL-like command-line interface for index management. 

This example aql script creates an integer index on `namespace` *user_profile*, `set-name` *west*,  and `bin-name` *last_activity*.

```bash
aql> CREATE INDEX ix1 ON user_profile.west (last_activity) NUMERIC
OK, 1 index added.
```

Once the index creation request reaches one node in the cluster, the Aerospike cluster propagates the request to all cluster nodes and begins background index creation on all nodes. Each cluster node can only manage secondary indexes for resident data. 

### Checking Index Status

To check the status of an index:

```bash
aql > show indexes
+----------------+-----------------+--------+----------+-------+-----------+------------+--------------+
| ns             | bins            | set    | num_bins | state | indexname | sync_state | type         |
+----------------+-----------------+--------+----------+-------+-----------+------------+--------------+
| "user_profile" | "last_activity" | "west" | 1        | "WO"  | "ix1"     | "synced"   | "INT SIGNED" |
+----------------+-----------------+--------+----------+-------+-----------+------------+--------------+
1 row in set (0.000 secs)
+----------------+-----------------+--------+----------+-------+-----------+------------+--------------+
| ns             | bins            | set    | num_bins | state | indexname | sync_state | type         |
+----------------+-----------------+--------+----------+-------+-----------+------------+--------------+
| "user_profile" | "last_activity" | "west" | 1        | "WO"  | "ix1"     | "synced"   | "INT SIGNED" |
+----------------+-----------------+--------+----------+-------+-----------+------------+--------------+
1 row in set (0.000 secs)
```

In this example, the command output indicates that:

- There are two nodes (two tables display) in the cluster.
- Both nodes show that index creation is in the write-only state (`WO`). 

{{#note}}
Nodes can only be queried when index is in read-write state (`RW`). If not all nodes are in the `RW` state and a query is made against the cluster, an error with no data is returned to the caller.
{{/note}}

### Inserting Data

This example aql script inserts a record in preparation for a query. 

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

See the [Java Client - Synchronous](/docs/client/java/usage/kvs/write.html) database write example for the Java equivalent.

{{#note}}
aql is not intended for use by applications. 
{{/note}}

### Range Query

To run a query for a range of *last_activity*:

```bash
aql> SELECT * FROM user_profile.west WHERE last_activity BETWEEN 340 AND 345
+----------+---------------+
| location | last_activity |
+----------+---------------+
| "AL"     | 340           |
| "MA"     | 342           |
| "CA"     | 345           |
| "AZ"     | 345           |
+----------+---------------+
4 rows in set (0.001 secs)
```

Range query result sets are *inclusive*; both specified values are included in the results (_340_ and _345_ in the above example).

If a cluster node goes down, in-progress queries return an error and partial results.

### Other Tools

Other ways of creating indexes are:

- [asinfo](/docs/tools/asinfo/index.html) tool sends raw info commands to the Aerospike cluster.
- `createIndex` and `dropIndex` equivalent Client API calls in each language binding.

{{#info}}
Aerospike does not support SQL as a query or management language.
{{/info}}

##References

See these topics for language-specific examples:

1. [Java](/docs/client/java/usage/query/query.html)
2. [C# .NET](/docs/client/csharp/usage/query/query.html)
3. [Node.js](/docs/client/nodejs/usage/query/query.html)
4. [Go](/docs/client/go/usage/query/query.html)
5. [Python](/docs/client/python/usage/query/query.html)
6. [PHP](/docs/client/php/usage/query/query.html)
7. [C](/docs/client/c/usage/query/query.html)
