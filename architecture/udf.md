---
title: User-Defined Functions
description: User-Defined Functions (UDFs) significantly extend the functionality and performance of the Aerospike database.
assets: /docs/architecture/assets
---

A User-Defined Function (UDF) is code written by a user (or developer) that runs inside the Aerospike database server. UDFs can significantly extend the capability of the Aerospike Database engine in functionality and in performance. Aerospike currently only supports [Lua](http://lua.org) as the UDF language.

{{#note}}
See the [UDF feature guide](/docs/guide/udf.html).
{{/note}}

### Why Lua?

Compared to other scripting languages, Lua is very simple and powerful.

{{#info}}
_Lua is a powerful, fast, lightweight, embeddable scripting language. Lua combines simple procedural syntax with powerful data description constructs based on associative arrays and extensible semantics._  &mdash; [Lua.org](http://www.lua.org/about.html)
{{/info}}

Lua has been used to great effect in a number of similar projects, such as arbitrary re-write rules within the Apache Squid proxy, UDFs in both Redis and Postgres, and other projects.

Aerospike creates a queue of Lua contexts&mdash;no more than one per thread, per registered UDF&mdash;greatly reducing latency and increasing performance.

### Record UDFs

Record UDFs execute on a single database record.They can create, update, or delete a record.

The Record UDF API provides:
- Access to the record object, including its bins and metadata (generation, TTL)
- Easy manipulation of complex data type such as Map and List
- Easy binary access to blob data types

Record UDFs execute in the transaction main flow. Every Record UDF is part of a flow that accesses a row in the database, creates the Lua Record object, and invokes the Lua function to allows the UDF to operate. Sometimes the record does not exist, which allows the UDF to create a record, and allows record manipulation.

{{#note}}
See the feature guide section on [Record UDFs](/docs/guide/record_udf.html).
{{/note}}

### Stream UDFs

Stream UDFs perform read-only operations on a collection of records. The Stream UDF system allows a _MapReduce_ style of programming, which is used for common, highly parallel MapReduce jobs such as _word count_&mdash;each row is accessed and a list of words and their counts are emitted. The top results are calculated in a reduce phase. For simple aggregations where counters are simply incremented in a context instead of continually creating and destroying objects, Aerospike provides optimal implementation.

An important difference between the Aerospike Stream UDF system and typical MapReduce frameworks is that the Aerospike Stream UDF is very low latency and has high reliability in a shared-nothing architecture.

Instead of coordinated job control, Stream UDF queries are sent to all cluster nodes by a requesting client. They are managed and prioritized by each server. Results return to the requesting client, which performs final operations (such as reduce or final aggregation) before returning the results to the client.

Although this can result in high memory use on the client, the server cluster is not disrupted by a poorly formed query. If lightweight app servers need to make requests that demand a large final reduce stage, you can add an intermediate application server to use a REST or SOA API. This acts like a coordinating agent, and that server can have a memory profile that fits the application load.

{{#note}}
See the feature guide section on [Stream UDFs](/docs/guide/stream_udf.html).
{{/note}}

### Managing UDF Code in a Cluster

UDFs are managed by the system metadata (SMD) component of the cluster. When a Lua module is registered, it is sent to only one node. That node forwards the request to the current cluster principal, who compares this incoming version with previous versions. If the version is newer, it persists the file to local storage (in a _user_ directory) and sends it to the remaining cluster nodes.

During registration, the file is interpreted. This allows immediate detection of simple parsing errors. Those errors are returned by the registration API.

{{#note}}
See [Managing UDFs](/docs/udf/managing_udfs.html).
{{/note}}

### Invoking UDFs
UDFs are organized into modules, where a module is a file containing Lua code
that was registered with the database. To invoke a UDF from the client the
developer will
1. Specify the UDF module name.
1. Specify the function name within the module to execute.
1. Specify arguments to the user-defined function (optional).

#### Multicore

Many other interpretive languages can only run one execution thread per process. For example, CPython uses globals in its code base, which greatly reduces the ability to run multiple Python contexts per process.

#### Gateway to C

Lua code can call C functions directly. Although the overhead to do so is measurable, it is simple. Aerospike provides examples of compiling and registering a shared C object, then calling one of the C functions directly from a Lua UDF module.

{{#note}}
See [Using C Functions in UDF Modules](/docs/udf/developing_lua_modules.html#using-c-functions-in-udf-modules).
{{/note}}

### Protection and Sandboxing

The Aerospike UDFs operate in-process. This maximizes performance, but poorly written UDFs can cause performance problems. Aerospike provides the ability to prevent similar errors in UDFs. Infinite loops are protected through limiting the amount of time spent in a UDF.

### Stored Procedures vs. UDFs

Stored procedures are commonly used in database systems. Stored procedures are similar to UDFs in that they are a user application program  stored and run in a Relational Database Management System (RDBMS). Stored procedures can read or write one or more records, and so are general-purpose mechanisms. 

UDFs are more limited. They operate either on a single record (**Record UDFs**) or a selected stream of records (**Stream UDFs**). Record UDFs behave like a traditional UDFs. Stream UDFs behave somewhat like stored procedures&mdash;they manage multiple records.
