---
title: Data Types
description: Aerospike bins can each hold a distinct scalar or complex data type
---
<div style="float: right" >
![]({{book.assets}}/icon-data-types2.png)
</div>

Aerospike [bins](/docs/architecture/data-model.html#bins) can each hold a distinct
[scalar](#basic-data-types) or [complex](#complex-data-types-cdts-) data type.

## Basic Data Types

Aerospike's scalar data types are:

 - integer
 - string
 - bytes
 - double
 - language-specific serialized blobs

### Integer

Integers are 64-bit numerics.
- Each each integer requires 8 bytes (64 bits) of storage.
- For touchpoints in the database where integers need to assume a signed-ness, such as secondary index, the integer is signed, in the range of [ −(2^63) to 2^63 − 1 ]. 
- An atomic increment operation can be applied to integer values. This can be used to implement counters.
- [Secondary indexes](/docs/architecture/secondary-index.html) are supported on integer data type, for both equality and range queries.

### String

Strings are opaque byte arrays. They are not bound to an encoding, which allows you to use any character encoding for strings.

- For touchpoints in the database where strings need to assume an encoding, it will assume UTF-8.
- Client libraries can enforce encoding (for example, Java and C# clients always convert to UTF-8). Refer to the client library for your implementation.
- Cross-language compatibility is up to the application. The application sets encoding for reads and writes reads.
- Record size is limited to 128 KB. Do not store huge strings.
- Atomic prepend and append operations can be applied to string values.
- [Secondary indexes](/docs/architecture/secondary-index.html) are supported on string data type, for equality query.

### Bytes

Bytes are byte arrays of a specific size.

- Any binary data of any type can be stored. Note that bytes are not NULL terminated.
- As of version 4.6.0, an extensive API of [bitwise operations](/docs/guide/bitwise.html) can be applied to byte values.

### Double

The double data type is stored in 64-bit IEEE-754 format.

- An atomic increment operation can be applied to double values.

{{#note}}
Aerospike versions 3.6.0 and later support double data types.
{{/note}}

## Complex Data Types (CDTs)

Aerospike's complex data types are:

- [Lists](/docs/guide/cdt-list.html)
- [Maps](/docs/guide/cdt-map.html)
- [Geospatial](/docs/guide/geospatial.html)

Each complex data type has an API of operations for atomically modifying its elements,
including in [deeply nested structures](/docs/guide/cdt-context.html),
as well as operations for querying its data.

{{#note}}
When moving from relational (or other NoSQL) databases to Aerospike, one common
concern is that only single-record transactions are supported in Aerospike.
Use cases, typically modelled across multiple tables in another database, can
instead be modeled with nested List and Map, and using their API methods in
multi-operation, single record transactions.
{{/note}}

### List

The [list](/docs/guide/cdt-list.html) data type is an ordered collection of scalar or CDT values.

The following example uses the [_aql_](/docs/tools/aql) tool to insert multiple values into a List.

```bash
aql> insert into test.demo (PK, bin) values ('key', 'JSON[1,2,3]')
OK, 1 record affected.

aql> select * from test.demo where PK='key'
+-----------+
| bin       |
+-----------+
| [1, 2, 3] |
+-----------+
1 row in set (0.000 secs)
```

### Map

The [map](/docs/guide/cdt-map.html) data type is a collection of distinct key-value pairs.
Map keys are any supported scalar data type. Map values can be any scalar data
type or have a CDT value nested within.

The following example uses the aql tool to insert values into a map bin.

```bash
aql> insert into test.demo (PK, bin) values ('key2', '{"str1":"ABC", "int1":26, "str2":"a map"}')
OK, 1 record affected.

aql>  select * from test.demo where PK='key2'
+---------------------------------------------+
| bin                                         |
+---------------------------------------------+
| "{"str1":"ABC", "int1":26, "str2":"a map"}" |
+---------------------------------------------+
1 row in set (0.002 secs)
```

Maps and lists allow arbitrary nesting, as shown in the examples below.

### Nesting Maps and Lists

This example shows how you can include list and map bins within the same record.

```bash
aql> insert into test.demo (PK, stringbin, intbin, listbin, mapbin) values ('key', 'a string', 10, 'JSON["list" ,"of" , "strings"]', '{"map":1, "of":2, "items":3}')
OK, 1 record affected.

aql>  select * from test.demo where PK='key'
+----------------------------------+------------+--------+-----------------------------------+--------------------------------+
| bin                              | stringbin  | intbin | listbin                           | mapbin                         |
+----------------------------------+------------+--------+-----------------------------------+--------------------------------+
| LIST('["str1", "str2", "str3"]') | "a string" | 10     | LIST('["list", "of", "strings"]') | "{"map":1, "of":2, "items":3}" |
+----------------------------------+------------+--------+-----------------------------------+--------------------------------+
1 row in set (0.001 secs)
```

This example shows arbitrary nesting with a Map nested within a List. 

```bash
aql> insert into test.demo (PK, stringbin, intbin, ListNestedMap) values ('key3', 'a string', 10, 'JSON["list" ,"of" , {"map":1, "of":2, "items":3}, "strings"]')
OK, 1 record affected.

aql>  select * from test.demo where PK='key3'
+------------+--------+-----------------------------------------------------------------+
| stringbin  | intbin | ListNestedMap                                                   |
+------------+--------+-----------------------------------------------------------------+
| "a string" | 10     | LIST('["list", "of", {"items":3, "of":2, "map":1}, "strings"]') |
+------------+--------+-----------------------------------------------------------------+
1 row in set (0.001 secs)

```

In this example, the JSON document is passed to the aql tool, which converts it into a list containing string, integer, list, and map values.

```bash
aql> insert into test.demo (PK, bin) values ('key', 'JSON["string", 10, ["list" ,"of" , "strings"], {"map":1, "of":2, "items":3}]')
OK, 1 record affected.

aql> select * from test.demo where PK='key'
+-------------------------------------------------------------------------+
| bin                                                                     |
+-------------------------------------------------------------------------+
| ["string", 10, ["list", "of", "strings"], {"items":3, "of":2, "map":1}] |
+-------------------------------------------------------------------------+
1 row in set (0.000 secs)
```

## GeoJSON

Aerospike supports GeoJSON data types, which are JSON formatted strings used to encode geospatial information. See [Geospatial Index and Query](/docs/guide/geospatial.html).

{{#note}}
Aerospike versions 3.7.0 and later support GeoJSON data types.
{{/note}}

## Language Specific Serialized Blobs

The following Aerospike clients serialize unsupported data types into a blob using their native-language serialization. These blobs get deserialized automatically on reads.

- Java
- C# .NET
- Node.js
- PHP
- Python
- Ruby
- C

