---
title: Examples
description: These source code examples demonstrate using the Aerospike C# client with the Aerospike database.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Source code examples that demonstrate Aerospike C# client usage are located in the _AerospikeDemo_ project, and include:

Example | Description
--- | ---
Server Info | Use Aerospike info protocol to query a server node for statistics.
Put/Get | Write and read a record.
Replace | Write bins with replace option.
Add | Perform a server integer add.
Append  | Perform a server string append.
Prepend | Perform a server string prepend.
Batch | Perform multiple exists, get, or get header commands in a single batch.
Generation  | Use record generation to verify that a record didn't change since the last read.
Serialize | Use C# runtime default serialization when writing and reading a bin.
Expire  | Set record expiration to limit its lifetime.
Touch | Extend the life of records about to expire.
Operate | Perform multiple operations on a single record in one database command.
Delete Bin  | Delete a bin in a record.
Scan Parallel | Scan all records in a namespace and set in parallel using one thread-per-server node.
Scan Series | Scan all records in a namespace and set by querying each server node in series.
Async PutGet  | Write and read a record in asynchronous mode.
Async Batch | Asynchronously perform multiple exists, get, or get header commands in a single batch.
Async Scan  | Asynchronously scan all records in a namespace and set by querying each server node in series.
List/Map  | Write and read records that contain combinations of List and Dictionary bins.
User Defined Function | Call UDFs on the server.
Large List | Perform large list collection operations on a single bin in a record.
Large Set | Perform large set collection operations on a single bin in a record.
Large Stack | Perform large stack collection operations on a single bin in a record.
Query Integer | Query bins using an integer index/filter.
Query String  | Query bins using a string index/filter.
Query Filter  | Query on a secondary index with a filter, and then apply an additional filter in the UDF.
Query Sum | Query records and calculate the sum with an aggregate UDF.
Query Average | Query records and calculate average with a aggregate UDF.
Query Execute | Run a UDF on records matching a query filter.

### Executing

1. Ensure that Aerospike server(s) are up and running.
1. Open _Aerospike.sln_ in Visual Studio.
1. Press **F5** to start the AerospikeDemo application.
1.  Enter the server host IP address and port (typically 3000) of a node in the Aerospike cluster at the top of the window.
1. Enter the **namespace** (default is _test_) configured on the cluster
1. Enter a **set** name.
1. In the left pane, select an example (for example, _ServerInfo_). 

	The example source code appears.
1. Click **Start**.

	The bottom half of the window displays the results generated by the example.

