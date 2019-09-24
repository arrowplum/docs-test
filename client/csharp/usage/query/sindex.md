---
title: Manage Indexes
description: Use the Aerospike C# client to create and delete secondary indexes from the Aerospike database.
---

Use the Aerospike C# client to create and delete secondary indexes from the Aerospike database.

### Creating a Secondary Index

{{#note}}
Secondary index creation and removal are expensive operations, and should only be performed as administrative tasks, not as runtime tasks of an application.
{{/note}}

Secondary indexes can only be created once on the server as a combination of namespace, set, and bin name with either integer or string data types. For example, if you define a secondary index to index bin _x_ that contains integer values, then only records containing bins named _x_ with integer values are indexed. Other records with a bin named _x_ that contain non-integer values are not indexed.

To create an index and waiting for the index creation to complete for index `idx_foo_bar_baz` on namespace _foo_ within set _bar_ and bin _baz_:

```cs
IndexTask task = client.CreateIndex(null, "foo", "bar", "idx_foo_bar_baz", "baz", IndexType.NUMERIC);
task.Wait();
```
When an index management call is made to any node in the Aerospike Server cluster, the information automatically propagates to the remaining nodes.

### Remove a Secondary Index

To remove a secondary index, invoke `AerospikeClient.DropIndex()`.


