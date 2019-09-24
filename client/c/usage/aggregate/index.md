---
title: Aggregation
description: Aggregation queries using the Aerospike C client and Aerospike database.
---

There are various forms of processing results of a [secondary index query](/docs/client/c/usage/query/sindex.html). A common form is aggregation, where you apply a function on the entire results set of a query.

Many developers use SQL for define aggregation queries against a database. For example, the following SQL statement counts rows from the database:

```sql
SELECT count(*)
FROM test.demo 
WHERE d = 50
```

This counts the number of records in _test.demo_ that contain a column _d_ equal _50_. In Aerospike you can use UDFs and queries to do this.

Read these sections before continuing:

- [Connecting](/docs/client/c/usage/connect) &mdash; Establishing cluster connections.
- [Manage Indexes](/docs/client/c/usage/query/sindex.html) &mdash; Creating secondary indexes.
- [Register UDF](/docs/client/c/usage/udf/register.html) &mdash; Registering UDFs.
- [Query Records](/docs/client/c/usage/query/query.html) &mdash; Creating and executing queries.

### Defining Query

Read [Query Records](/docs/client/c/usage/query/query.html) to learn how define a query:

```cpp
as_query query;
as_query_init(&query, "test", "demoset");
 
as_query_where_inita(&query, 1);
as_query_where(&query, "d", integer_equals(50));
 
as_query_apply(&query, "mymodule", "mycount", NULL);
```

Use this `query` object to query for records in the Namespace _test_ within set _demoset_, looking for records with bin name _d_ and an integer value of _50_. The `mycount()` Stream UDF is applied on the result set of the query. (Stream UDFs are in the _mymodule_ UDF module.)

{{#note}}
Queries without a "where" clause result in a full database scan.
{{/note}}

### Defining a Stream UDF

The `mycount()` Stream UDF allows you to process a stream of data:

```lua
local function one(rec)
    return 1
end
 
local function add(a, b)
    return a + b
end
 
function mycount(stream)
    return stream : map(one) : reduce(add);
end
```

The `mycount()` Stream UDF is applied to a stream of results from the query. We can add to the stream these operations to perform on the results:

- `map` &mdash; Maps a value from the stream to another value. In this example, mapping is defined as `one()`, which maps a record to the value 1.
- `reduce` &mdash; Reduces the values from the stream to a single value. In the example, reduction is performed by adding two values from the stream, which are the 1s from `map`.

The end result is a stream that contains a single value: the `count` (or the sum of 1 for each record in the result set).

### Registering UDFs

Before making a query using the Stream UDF, the UDF must register with the Aerospike server.

```cpp
as_error err;

if (aerospike_udf_put(&as, &err, NULL, "mymodule", AS_UDF_TYPE_LUA,
        &udf_content) != AEROSPIKE_OK) {
    LOG("aerospike_udf_put() returned %d - %s", err.code, err.message);
}

```

### Executing Queries

To execute the query using `aerospike_query_foreach()`:

```cpp
if (aerospike_query_foreach(&as, &err, NULL, &query, each_value, NULL) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

### Processing the Results

Call `each_value()` for each value that returns from the query:

```cpp
bool each_value(const as_val *val, void *udata) {
    if (val == NULL) {
        // query is complete
        return true;
    }
 
    as_integer *ival = as_integer_fromval(val);
  
    if (ival == NULL) {
        // abort the query
        return false;
    }
 
    // process the value
 
    return true;
}
```
The example above returns a single integer: the count of the records that satify the query.

To pass a global object each time during the callback, provide `userdata`in `as_query_foreach()`.

### Cleaning Up Resources

After the query results complete processing, the client can safely destroy the query object and its member objects using `as_query_destroy()`. Note the example avoids an explicit `as_query_destroy()` by using the stack-allocated `as_query` object and `as_query_where_inita()` to avoid using the internal heap.

