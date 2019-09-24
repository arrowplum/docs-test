---
title: Examples
description: Use the Aerospike Ruby client to develop Ruby applications to store and retrieve data in the Aerospike database.
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
---

Use the Aerospike Ruby client to develop Ruby applications to store and retrieve data in the Aerospike database.

 Aerospike Ruby client usage examples are located in the *examples* directory in **<a id="github" href="http://github.com/aerospike/aerospike-client-ruby">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a>**.

Examples include the following.

Example | Description
--- | ---
Server Info | Use the Aerospike info protocol to query a server node for statistics.
Put/Get | Write and read a record.
Replace | Write bins using the `replace` option.
Add | Perform a server integer add.
Append  | Perform a server string append.
Prepend | Perform a server string prepend.
Batch | Perform multiple exists, get, or get header commands in a single batch.
Generation  | Verify that a record didn't change since the last read.
Expire  | Set record expiration.
Touch | Extend the life of records about to expire.
Operate | Perform multiple operations on a single record in one database command.
Delete Bin  | Delete a record bin.
Scan Parallel | Scan all records in a specified namespace and set in parallel, using one thread-per-server node.
Scan Series | Scan all records in a specified namespace and set by querying each server node in series.
List/Map  | Write and read records that contain a combination of List and Dictionary bins.
User Defined Function | Call user-defined functions on the server.
GeoJSON | Use geospatial index to query for records using points-within-region and region-contains-point filters.

### Usage

Run one or run all examples in one sweep:

```bash
$ cd examples
$ ruby <name> -h <host> -p <port> -n <namespace> -s <set>
``` 
 
To touch an example in namespace *test* and set *demoset*:

```bash
$ ruby examples/touch.rb -h localhost -p 3000 -n test -s demoset
```

