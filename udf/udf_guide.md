---
title: User-Defined Functions (UDF) Development Guide
description: Learn about how you can leverage Aerospike User-Defined Functions to specify server-side function rather than on the client side to drastically reduce client/server calls.
assets: /docs/udf/assets
---

### Introduction

An Aerospike [User-Defined Function](/docs/guide/udf.html) is a piece of code, written by a developer in [Lua programming language](http://www.lua.org/about.html) (or C called from Lua) that runs inside the Aerospike database server.

There are two types of UDFs in Aerospike: Record UDFs and Stream UDFs.
A Record UDF operates on a single record.  A Stream UDF operates on a stream of records, and it can comprise multiple stream operators to perform very complex queries.

The complexity of UDFs can range from a single function that is only a few lines long, to a multi-thousand line module that contains many internal functions and multiple external functions.

When contemplating the construction of a new UDF, it is important to consider
the application data model and the interaction between record bins.
The general pattern (or life-cycle) for UDF development is:

* Design the application data model
* Design the UDFs to perform desired functions on the data model
* Create/Test the UDFs
* Register UDFs with the Aerospike database
* Iterate function test/development cycle
* System Test
* System Deployment

UDF design and development is usually an iterative activity, where the first
version is simple and then, potentially, evolves over time to something complex.

### Developing UDFs in Aerospike

Probably the best way to get famililar with the UDF mechanism is to create
the "Hello World" Record UDF. 
Following the instructions below, you will see how to build, register and execute your "Hello World" UDF. 

```
function hello_world(rec)
  return "Hello World!!"
end
```

This "Hello World" Record UDF is invoked when a specific record is referenced,
and it simply returns the "Hello World!!" string to the caller.

In general, the section
[Developing UDF Modules](/docs/udf/developing_lua_modules.html) covers the
preparation of a UDF and the section
[Managing UDFs Guide](/docs/udf/managing_udfs.html) covers the
registration and execution the UDF.

### Developing Record-Based UDFs

A Record UDF is invoked once for each record in the Aerospike Key-Value
operation. 
Typically, excluding batch operations, only a single record is the
target of a KV operation. 
In general, a Record UDF can do the following:

* Create/delete bins in the record.
* Read any/all bins of the record.
* Modify any/all bins of the record.
* Delete/Create the specified record.
* Access parameters constructed and passed in from the client.
* Construct results as any server data type (e.g. string, number, list, map) to be returned over the wire to the client.

#### Example: Record Create or Update

In this simple example
([ Annotated Record UDF ](/docs/udf/examples/record_udf_annotated.html)),
we show how the UDF can create or update an Aerospike record.

In addition UDF developers can use the following Aerospike commands
( Lua: aerospike Module ) to perform record operations:

```
status = aerospike:create( rec )
status = aerospike:exists( rec )
status = aerospike:update( rec )
status = aerospike:remove( rec )
```

#### Example: Basic Statistics Management

In this moderately complex example
( [ Statistics Record UDF ](/docs/udf/examples/record_udf_statistics.html)),
we show how a UDF can manage some numerical statistics
(Max Value, Min Value, Ave Value, Count) of values that are kept in
various KV Record bins. 

In addition, in this example we make use of the Lua: logging Module functions
(which are explained in more detail in the UDF
[ UDF Best Practices Guide ](/docs/udf/best_practices.html)).
The logging functions write messages to the log or the console:

```
info(message)
debug(message)
trace(message)
warn(message)
```

These log functions will generate output in the log or console.  The following Lua line:

```
trace("[ENTER]<%s>  Value(%s) valType(%s)",  meth, tostring(newValue), type(newValue));
```

will generate output as follows:

```
Sep 21 2013 22:25:22 GMT: DETAIL (ldt): (/home/aerospike/src/lua/udf_samples.lua:160) [ENTER]<unique_set_write()> Value(Map("B"->71, "A"->70)) valType(userdata)
```

The log functions are controlled by the log/console stanzas in the Aerospike server config file.

### Aerospike Client Record UDF Invocation

The following language-specific links provide information on invoking Record UDFs from respective Aerospike Client.

* [Java](/docs/client/java/usage/udf/apply.html)
* [C# .NET](/docs/client/csharp/usage/udf/apply.html)
* [Node.js](/docs/client/nodejs/usage/udf/apply.html)
* [PHP](/docs/client/php/usage/udf/apply.html)
* [Python](/docs/client/python/usage/udf/apply.html)
* [Go](/docs/client/go/usage/udf/apply.html)
* [Ruby](/docs/client/ruby/usage/udf/apply.html)
* [C](/docs/client/c/usage/udf/apply.html)

### Record UDF: More Detail

[ Record UDF Development ](/docs/udf/developing_record_udfs.html) section provides more details on Record UDF development. And [ Record UDF Examples ](/docs/udf/examples/record_udf_examples.html) show several Record UDFs that we've developed for both testing and documentation purposes.

### Stream UDF

A Stream UDF is invoked on a stream of records rather than a single record.

Using secondary indexes on bins in a record, a subset of records matching the
query criteria can be streamed out. A Stream UDF can be used to extract values in bins of records, get count of records or similar statistics on this extracted stream of records.

The process can be summarized as:

* Create a Secondary Index
* Run a Query on a Secondary Index
* Apply stream UDF on results of a secondary index query.

The following language-specific links provide information on using Stream UDFs from respective Aerospike Client.

* [Java](/docs/client/java/usage/aggregate)
* [C# .NET](/docs/client/csharp/usage/query/aggregate.html)
* [Node.js](/docs/client/nodejs/usage/query/aggregate.html)
* [PHP](/docs/client/php/usage/udf/aggregate.html)
* [Python](/docs/client/python/usage/udf/aggregate.html)
* [C](/docs/client/c/usage/query/aggregate.html)

#### Example: Simple Statistics

In this example, [ Stream UDF – Simple Statistics](/docs/udf/examples/stream_udf_stats.html), we calculate simple statistics information on a data set.

#### Example: Word Count

In this example, [ Stream UDF - Word Count ](/docs/udf/examples/stream_udf_word_count.html), we count the number of times each word appears in a book.

### Stream UDF: More Detail

[ Stream UDF Development ](/docs/udf/developing_stream_udfs.html) section provides more details on Stream UDF development. And [ Stream UDF Examples ](/docs/udf/examples/stream_udf_examples.html) show several Stream UDFs that we've developed for both testing and documentation purposes.

### Functional Benefits of UDFs

When contemplating the use of a UDF, make sure to first review the extensive
atomic operations for the [List](/docs/guide/cdt-list.html) and
[Map](/docs/guide/cdt-map.html) data types.  Multiple operations can be chained
to execute in order on a single record, using the
[operate()](/docs/client/java/usage/kvs/multiops.html) command of the Aerospike client.

Such single record transactions are faster and scale better than UDFs. However,
there are several situations where a UDF may be advantageous,
compared to implementing similar functionality on the application side.

* Extending the functionality of the [complex data types](/docs/guide/data-types.html#complex-data-types-cdts-).
with new atomic operations, or to implement new collection types (see Record UDF examples).
* A background UDF, a record UDF applied to multiple records via scan or query,
  can be used to carry out maintenance.
* Implementing aggregate functions using a stream UDF.

### More Information

* [Knowing Lua](/docs/udf/knowing_lua.html) : An introduction to using Lua.
* [Developing UDF Modules](/docs/udf/developing_lua_modules.html) : Details on how to create Lua Modules for running within an Aerospike database.
* [Developing Record UDFs](/docs/udf/developing_record_udfs.html) : An introduction to developing a Record UDF in Lua.
* [Developing Stream UDFs](/docs/udf/developing_stream_udfs.html) : An introduction to developing a Stream UDF in Lua.
* [Managing UDFs](/docs/udf/managing_udfs.html) : How to install, remove, update UDF modules, and managing runtime UDF caches.
* [Known Limitations](/docs/udf/known_limitations.html) : Known limitations of the UDF system.
* [Best Practices](/docs/udf/best_practices.html) : Tips for improving the developer experience while developing Lua in Aerospike.
* [API Reference](/docs/udf/api_reference.html) : The API Reference for Aerospike extensions to Lua, including functions, modules, types and Large Types.
* [Record UDF Examples](/docs/udf/examples/record_udf_examples.html) : Useful and example Record UDFs.
* [Stream UDF Examples](/docs/udf/examples/stream_udf_examples.html) : Useful and example Stream UDFs.
