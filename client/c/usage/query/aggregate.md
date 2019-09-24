---
title: Aggregate Records
description: Use the Aerospike C client APIs to aggregate records to process query results.
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

- [Connect](/docs/client/c/usage/connect) &mdash; Establishing cluster connections.
- [Manage Index](/docs/client/c/usage/query/sindex.html) &mdash; Creating secondary indexes.
- [Register UDF](/docs/client/c/usage/udf/register.html) &mdash; Registering UDFs.
- [Query](/docs/client/c/usage/query/query.html) &mdash; Creating and executing queries.

### Defining Query

Read [Query Records](/docs/client/c/usage/query/query.html) to learn how define a query:

```cpp
as_query query;
as_query_init(&query, "test", "demoset");
 
as_query_where_inita(&query, 1);
as_query_where(&query, "d", integer_equals(50));
 
as_query_apply(&query, "mymodule", "mycount", NULL);
```
This query returns all records in the namespace _test_ and sets _demoset_, where the bin _d_ contains the value  _50_. The `mycount()` Stream UDF in the UDF module `mymodule` is then applied on the query results.

{{#note}}
A query without a "where" clause results in a full database scan.
{{/note}}

### Defining Stream UDFs

To count the number of records that satisfy the query, define a UDF to count all returned records. When a query executes, it produces a stream of results. [Query Records](/docs/client/c/usage/query/query.html) explains that the stream contains records that you can iterate using the client API. However, Aerospike provides the ability to process the stream of results using a **Stream UDF**. Stream UDFs allow a stream to be augmented with operations that process the data flowing through the stream.

This example defines a Stream UDF _mycount_ defined in `mymodule` to process a stream of data:

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

`mycount()` is applied to a stream (in this case, the stream of results from the query). We add to the stream the operations we want to perform on the results:

- `map` &mdash; Maps a value from the stream to another value. In the example, mapping is defined as the function `one()`, which maps a record to the value 1.
- `reduce` &mdash; Reduces the values from the stream into a single value. In the example, reduction is performed by adding two values from the stream, which happen to be 1s returned from the `map` function.

The end result is a stream that contains a single value: the _count_ or more technically, the sum of 1 for each record in the result set.

### Registering UDFs

The UDF module must be registered with the Aerospike server before you can use a Stream UDF. See [Registering UDFs](/docs/client/c/usage/udf/register.html). The following example registers `mymodule` with the server:

```cpp
as_error err;

if (aerospike_udf_put(&as, &err, NULL, "mymodule", AS_UDF_TYPE_LUA,
        &udf_content) != AEROSPIKE_OK) {
    LOG("aerospike_udf_put() returned %d - %s", err.code, err.message);
}

```

### Executing the Query

To execute the query and process the results, use `aerospike_query_foreach()`"

```cpp
if (aerospike_query_foreach(&as, &err, NULL, &query, each_value, NULL) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

### Processing the Results

`each_value()` is called for each value that returns from the query:

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

This returns a single integer: the count of the records that satify the query condition.

To pass a global object for each value during a callback, add the `userdata` parameter in `as_query_foreach()`.

### Cleaning Up Resources

After the query results complete processing, the client can safely destroy the query object and its member objects using `as_query_destroy()`. Note the example avoids an explicit `as_query_destroy()` by using the stack-allocated `as_query` object and `as_query_where_inita()` to avoid using the internal heap.


