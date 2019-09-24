---
title: Query Records
description: Use the Aerospike C client to query on secondary indexes.
---

To use the Aerospike C client to query on secondary indexes, initialize and populate an `as_query` object, and then execute the query using `aerospike_query_foreach()`. 

A query can:

- Make a callback that returns each record satisfying the query.
- Make a callback that returns the result of applying a [Stream UDF](/docs/client/c/usage/query/aggregate.html) on the set of records resulting from query.

See [Aggregate](/docs/client/c/usage/query/aggregate.html) for information on Stream UDFs.

These examples are excerpts from _examples/query\_examples/simple_ in the Aerospike C client package.

See [Manage Indexes](/docs/client/c/usage/query/sindex.html) for information on secondary indexes.

To continue, the client must have an [established the cluster connection](/docs/client/c/usage/connect).

### Defining the Query

To initialize and build a query object to find records in bin _test-bin_, in the namespace _test_ within set _demoset_ that contain the integer value 7:

```cpp
as_query query;
as_query_init(&query, "test", "demoset");

as_query_where_inita(&query, 1);
as_query_where(&query, "test-bin", integer_equals(7));

```
The equivalent SQL clause is: `select * from test.demoset where test-bin equal 7`.

{{#note}}
A query without a `where` clause results in a full database scan.
{{/note}}

### Invoking the Query

To execute the query, invoke `aerospike_query_foreach()`:

```cpp
if (aerospike_query_foreach(&as, &err, NULL, &query, query_cb, NULL) !=
        AEROSPIKE_OK) {
    LOG("aerospike_query_foreach() returned %d - %s", err.code, err.message);
    as_query_destroy(&query);
    cleanup(&as);
    exit(-1);
}
```

`aerospike_query_foreach()` initiates one query for each node in the cluster. The Aerospike C client, internally has five `NUM_QUERY_THREADS` query worker threads processing all query jobs for all nodes. 

{{#note}}
If your cluster has more than five nodes, increase the default thread count to allow full node parallelism.
{{/note}}

- If no secondary index was created on the specified query object, an `AEROSPIKE_ERR_INDEX_NOT_FOUND` error returns.
- If an index creation was initiated but did not complete and then a query executes, an `AEROSPIKE_ERR_INDEX_NOT_READABLE` error returns.

### Processing Results

`aerospike_query_foreach()` takes `query_cb` as a parameter of type:

```cpp
typedef bool (*aerospike_query_foreach_callback)(const as_val *value, void *udata);
```

`aerospike_query_foreach()` is called for each record that returns from any node in no particular order. 

Because `value` is a const `as_val *`, the callback is not responsible for destroying `value`, and `value` is only available within the scope of the callback. The callback should not send `value` outside the scope of the callback.

The callback is called with a NULL `value` when there are no more results.

When `value` is expected to be a record, cast the value into a record using `as_record_fromval()`:
- If `value` is a record, then the function returns the record. 
- If `value` is not a record, then NULL returns. 

You can also use `as_val_type()` to check the `value` data type.

```cpp
bool callback(const as_val *value, void *udata) {
    if (value == NULL) {
        // query is complete
        return true;
    }
  
    as_record *rec = as_record_fromval(value);
  
    if (rec != NULL) {
        // process record
    }
 
    return true;
}
```

To pass a `global` object each time during the callback, place a `userdata` parameter in `as_query_foreach()`.

### Cleaning Up Resources

On query completion, use `as_query_destroy()` to safely destroy the `query` object and its member objects.

The example does not use an explicit `as_query_destroy()`, but uses the stack-allocated `as_query` object and  `as_query_where_inita()`, which avoids internal heap usage.


