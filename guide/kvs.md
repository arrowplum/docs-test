---
title: Key-Value Store
description: Aerospike is a distributed database supporting key-value store and document-oriented data models
assets: /docs/guide/assets
---
<div style="float: right" >
![]({{book.assets}}/icon-key-val2.png)
</div>

Aerospike is a row-oriented database, where every _record_ (equivalent to
a row in a relational database) is uniquely identified by a key. The record's
key and its other metadata live in the primary index. The record's data lives in
the pre-defined storage device of the namespace it occupies. For a more detailed
description read the [Data Model](/docs/architecture/data-model.html) section of
the architecture overview.


Application developers should keep these key things in mind:

- Aerospike is _schemaless_. You don't have to define _sets_ (equivalent to a table in a relational database) or _bins_ (equivalent to a column in a relational database) in advance. Sets and bins are created in realtime by the application as it performs write operations into Aerospike.
- Aerospike supports a key-value store model. Records are uniquely identified by a primary key composed of the _set_ and _userKey_ (the primary app-side identifier).
- Aerospike also supports a document store model. A single record can have multiple bins of different [data types](/docs/guide/data-types.html), including deeply nested [maps](/docs/guide/cdt-map.html) and [lists](/docs/guide/cdt-list.html) structures.
- Aerospike data types have APIs of atomic write operations and server-side read operations (such as map's [`get_by_rank_range()`](/docs/guide/cdt-map.html#map-apis).
- Starting with Aerospike version 4.6, map and list operations can be [applied on deeply nested elements](/docs/guide/cdt-context.html).
- Multiple operations on one or more bins can be combined into single record transactions (the **operate()** method in the clients). The transaction executes atomically under a record lock with isolation and durability.
- For non-atomic operations, the record's generation metadata (data modification count) can be used to ensure that the data being written was not modified since the last read. This optimistic concurrency allows application developers to use a _check-and-set_ (read-modify-write) pattern. For a language-specific example, see [Java Read-Modify-Write](/docs/client/java/usage/kvs/write.html#read-modify-write).

