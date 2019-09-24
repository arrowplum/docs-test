---
title: Managing Secondary Indexes
description: Use the Aerospike C client to create and delete secondary indexes from the Aerospike database.
---

Use the Aerospike C client to create and delete secondary indexes from the Aerospike database.

To continue, the client must have an [established the cluster connection](/docs/client/c/usage/connect).

These examples are excerpts from _examples/query_examples_ in the Aerospike C client package.

### Creating a Secondary Index

Secondary indexes can only be created once on the server as a combination of namespace, set, and bin name with either integer or string data types. For example, if you define a secondary index to index bin _x_ that contains integer values, then only records containing bins named _x_ with integer values are indexed. Other records with a bin named _x_ that contain non-integer values are not indexed.

When an index management call is made to any node in the Aerospike Server cluster, the information automatically propagates to the remaining nodes.

Secondary index creation and removal are expensive operations, and should be performed as administrative tasks, and not as runtime tasks of an application. Aerospike provides a number of tools such as `aql` to create, remove, manage, and monitor secondary indexes. The APIs here are provided to enable the building of these and other tools.

Use these operations to create secondary indexes, and specify the namespace, set, bin, and a secondary index name  to uniquely identify it to the namespace:

- `aerospike_index_integer_create()` &mdash; Create an index on bins with integer values.
- `aerospike_index_string_create()` &mdash; Create an index on bins with string values.

To create an integer index on the _binX_ bin, for records in namespace _test_ within set _demoset_ with the secondary index name _idx\_binX_:

```cpp
as_error err;

if (aerospike_index_integer_create(&as, &err, NULL, "test", "demoset",
		"binX", "idx_binX") != AEROSPIKE_OK) {
	LOG("aerospike_index_integer_create() returned %d - %s", err.code,
			err.message);
	return false;
}
```

### Removing a Secondary Index

Remove secondary indexes using `aerospike_index_remove()`, which takes the namespace and secondary index name.

```cpp
as_error err;

aerospike_index_remove(&as, &err, NULL, "test", "idx_binX");
```

