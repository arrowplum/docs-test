---
title: Single Record
description: Aerospike provides simple interfaces for reading data from and writing data to an Aerospike cluster. 
assets: /docs/guide/assets
---

Aerospike provides simple interfaces for reading data from and writing data to an Aerospike cluster. 

These are the supported key-value store operations:

| Operation | Description |
| --- | --- |
| `set` | Write any record, existing or not. |
| `create` | If the record does not exist, create the record and add the specified bins. |
| `update` | If the record exists, add or update the specified bins. Unspecified bins are unaltered. |
| `replace` | If the record exists, write the specified bins and replace all existing bins. |
| `delete` | Remove the record from the database. |
| `get` | Read the record. Specific bins can be projected. |
| `add` | Add (or subtract) a value to an integer bin. |
| `append` | Append a value to a string or byte array bin. |

## Code Examples

These examples create, get, update, and delete a record using the Aerospike [aql](/docs/tools/aql/index.html) tool.

To create a new record: 

```bash
aql> select * from test.demo where PK = 'key'
Error: (602) AEROSPIKE_ERR_RECORD_NOT_FOUND

aql> insert into test.demo(PK, bin) values ('key' , 'Bin Value')
OK, 1 record affected.

aql> select * from test.demo where PK = 'key'
+-------------+
| bin         |
+-------------+
| "Bin Value" |
+-------------+
1 row in set (0.000 secs)
```

To update an existing record:

```bash
aql> insert into test.demo (PK, bin) values ('key', 'Updated Bin Value')
OK, 1 record affected.

aql> select * from test.demo where PK = 'key'
+---------------------+
| bin                 |
+---------------------+
| "Updated Bin Value" |
+---------------------+
1 row in set (0.000 secs)
```

To create new record _key1_ in _bin_ with an integer value of 1 (different from the existing record): 

```bash
aql> insert into test.demo (PK, bin) values ('key1', 1)
OK, 1 record affected.

aql> select * from test.demo where PK = 'key1'
+-----+
| bin |
+-----+
| 1   |
+-----+
1 row in set (0.000 secs)

aql> select * from test.demo
+---------------------+
| bin                 |
+---------------------+
| 1                   |
| "Updated Bin Value" |
+---------------------+
2 rows in set (0.016 secs)
```

To delete the _key_ record:

```bash
aql> delete from test.demo where PK='key'
OK, 1 record affected.

aql> select * from test.demo
+-----+
| bin |
+-----+
| 1   |
+-----+
1 rows in set (0.010 secs)
```

All `get()` (read) and `set()` (write) operations take these additional parameters:

- transaction timeout&mdash;Presice time out. Trumps the retry policy.
- retry policy&mdash;Retry failed operations until the specified timeout.

`set()` (write) operations take an optional time-to-live (TTL) parameter that specifies how long to protect a record from automatic eviction by the system. Set TTL to infinite to specify never to evict that data.

## Document Store Model
Nested [lists](/docs/guide/cdt-list.html) and [maps](/docs/guide/cdt-map.html)
may be combined to model complex use cases.

Developers familiar with other NoSQL document stores will see the flexibility in
applying map and list operations on bins with nested CDTs. Combined with
(operate) single record transactions, Aerospike provides powerful functionality
for modeling a wide range of use cases.

## References

See these topics for language-specific examples:

- [Java](/docs/client/java/usage/kvs/write.html)
- [C# .NET](/docs/client/csharp/usage/kvs/write.html)
- [Node.js](/docs/client/nodejs/usage/kvs/write.html)
- [Go](/docs/client/go/usage/kvs/write.html)
- [Python](/docs/client/python/usage/kvs/write.html)
- [PHP](/docs/client/php/usage/kvs/write.html)
- [C](/docs/client/c/usage/kvs/write.html)

