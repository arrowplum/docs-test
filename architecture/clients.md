---
title: Client Architecture
description: The Aerospike Smart Client™ implements the APIs exposed to the transaction, making cluster membership changes transparent.
assets: /docs/architecture/assets
---

<div style="float: right" >
{{#figure "" "Aerospike client-server architecture" size="large"}}
![]({{book.assets}}/ARCH_user_mw_as_small.png)
{{/figure}}
</div>

This illustrates the Aerospike client-server architecture.

Often, stateless applications reside on a separate set of servers. To communicate with the Aerospike cluster, Aerospike provides a set of database clients&mdash;_drivers_. These clients provide cluster-status sensing, efficient transaction routing and network connection pooling, and failover protection. 

The client libraries are distributed (as source) to build in to your application server tier. They provide functionality so that a load balancer tier is not required to run with high availability.

When you compile your application with an Aerospike Client API library (such as the [C Client](/docs/client/c) or [Java Client](/docs/client/java)), it includes the Aerospike **Smart Client™**, which uses a binary wire protocol to connect to servers in a cluster. 

## Client Library Capabilities

These Aerospike Client API capabilities provide high performance and easy development:

- Tracks cluster states 
 - At any instant, the client uses the `info` protocol to communicate with the cluster and maintain a list of cluster nodes. 
 - The Aerospike **Smart Partitions™** algorithm determines which node stores a particular data partition. 
 - Automatically tracks cluster size changes entirely transparent to the application to ensure that transactions do not fail during transitions, and applications do not need to restart on node arrival and departure. 
- Implements connection pooling
 - You don't have to code, configure, or manage a pool of connections for the cluster.
- Manages transactions, monitors timeouts, and retransmits requests
- Thread safe 
 - Only one instance is required in a process.

{{#figure "" "Client architecture" size="large"}}
![]({{book.assets}}/architecture-client-small.png)
{{/figure}}

Besides basic `put()` `get()` and `delete()` operations, Aerospike supports:
- **CAS** (safe read-modify-write) operations.
- In-database counters.
- Batch `get()` operations.
- Scan operations.
- List and Map element operations such as `removeByKey()` or `getByValueRange()`.
- Queries&mdash;Bin values are indexed and the database searched by equality or range.
- User-Defined Functions (UDFs) extend database processing by executing application code in Aerospike.
- Aggregation&mdash;Use UDFs on a collection of records to return aggregate values.

## Client Feature Matrix

The [Client Feature Matrix](/docs/guide/client_matrix.html) contains up-to-date client library features.

