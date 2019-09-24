---
title: User-Defined Functions
description: Use Aerospike User-Defined Functions (UDFs) to extend database functionality and performance.
assets: /docs/guide/assets
---
<div style="float: right" >
![]({{book.assets}}/icon-udf.png)
</div>

<br>
A User-Defined Function (UDF) is code written by the application developer that runs inside the Aerospike database server. UDFs are intended to extend database functionality.

Aerospike currently supports [Lua](http://lua.org) alone as a UDF language.

## UDF Types

Aerospike has two kinds of UDF:

- **[Record UDFs](/docs/guide/record_udf.html)**
- **[Stream UDFs](/docs/guide/stream_udf.html)**

### Record UDFs

Record UDF are single functions that execute on single records, and can both
read and write data in the record.

Common uses for Record UDFs are:
- Implementing atomic operations that do not currently exist in the client.
- Extending the [predicate filtering](/docs/guide/predicate.html) operators for scan and query.
- Applying large scale maintenance to the database through a background UDF.

### Stream UDFs

Stream UDFs perform distributed stream processing, including distributed stream aggregation&mdash;where a stream of records returned from a query can be filtered, transformed, and aggregated on each node in the cluster and again by the client. 

Common uses for Stream UDFs are:
- Transforming query results.
- Aggregating query results.

## Managing UDFs

To invoke UDFs in your application:

1. Specify the UDF file name.
1. Specify the function name within the file
1. (optional) Specify one or more parameters.

Use the Aerospike [aql](/docs/tools/aql/udf_management.html) tool, and client APIs to manage UDFs in the cluster. 

## Registering UDFs

<div style="float: right" >
![]({{book.assets}}/UDF_Register.png)
</div>

UDFs must register with the Aerospike cluster before calling functions. UDFs only have to register with one node in the cluster. The Aerospike system metadata component distributes a copy to all other cluster nodes.

To replace a UDF file, you must register the new file. The server uses the latest version. 

## Writing UDFs

Develop UDFs using a single server node. This is because when testing a cluster, it is difficult to determine which node executed the function, and so hard to tell which log file to examine. 

### Logs

The Aerospike UDF logging feature routes logs to the Aerospike log file. 

### Error Detection

Syntax errors detected during registration return an error with the cause and line number where the error occurs. 

Runtime errors during execution such as an uncaught exception, return to the caller as a Lua error string and line number where the error occurs.

## References

- [UDF Developer Guide](/docs/udf/udf_guide.html)
- [UDF Architecture Guide](/docs/architecture/udf.html)

