---
title: Aggregate Records
description: Use the Aerospike Nodejs client to aggregate records and process the results of a query.
---

There are various forms of processing results of a [secondary index query](/docs/guide/query.html#secondary-index). A common form is aggregation, where you apply a function on the entire results set of a query.

Developers familiar with SQL aggregation queries will recognize the following SQL statement for counting rows from the database:

```sql
SELECT count(*) FROM test.demo WHERE d = 50
```

The statement counts the number of records in _test.demo_, containing column _d_ equal "50". In Aerospike you use a query and UDFs to perform this operation.

Read the following topics before continuing:

- **[Connecting](/docs/client/nodejs/usage/connect)** &mdash; Establish a cluster connection.
- **[Manage Indexes](/docs/client/nodejs/usage/query/sindex.html)** &mdash; Create a secondary index.
- **[Register UDF](/docs/client/nodejs/usage/udf/manage.html#registering-a-udf-module)** &mdash; Register a user-defined function.
- **[Query Records](/docs/client/nodejs/usage/query/query.html)** &mdash; Create and execute queries.


### Defining the Query

[Query Records](/docs/client/nodejs/usage/query/query.html) described defining a query. 

To define the query:

```js
var query = client.query('test', 'demo')
query.select('baz')
query.where(Aerospike.filter.range('baz', 1, 100))
```

This query returns all records in namespace *test* within set *demo* where the bin *baz* contains a value between 0 and 100.

### Defining the Stream UDF

To count the number of records that satisfy the query, define a UDF. 

When the query executes, it produces a stream of results. The stream contains records, which you can iterate over using the client API. However, Aerospike processes this stream of results using a **Stream UDF**. The stream can be augmented with operations to process the data flowing through the stream.

This example defines a Stream UDF _count_ in the *example.lua* module:

```lua
local function one(rec)
    return 1;
end

local function add(a, b)
    return a + b;
end

function count(stream)
    return stream : map(one) : reduce(add);
end
```

`count()` is a Stream UDF to apply to the stream. In this case, apply the following operations:

- `map` &mdash; Map a value from the stream to another value. In the example, mapping is defined as `one()`, which maps a record to the value 1.
- `reduce` &mdash; Reduce the values from the stream to a single value. In the example, reduction is performed by adding two values from the stream, which are the 1s from `map`.
The end result is a single value, *count*, or a sum of 1 for each record in the result set.

### Registering the UDF

The UDF module must register with the cluster (see [Registering a UDF Module](/docs/client/nodejs/usage/udf/manage.html#registering-a-udf-module)). 

This example registers the  UDF module _example.lua_:

```js

client.udfRegister('udf/example.lua', function (error) {
  if (!error) {
    // registration successfull
  }
})
```

### Configuring the UDF Search Path

Since some Stream UDF processing occurs on the client-side, the client must point to the local UDF module location. 

To define this location when connecting to Aerospike cluster use `config.modlua.userPath`:

```js
config = {
  modlua: {
    systemPath: '/opt/aerospike/udf/sys/',
    userPath: './udf'
  }
}
Aerospike.connect(config, function (error, client) {
})
```

### Executing the Query

To execute the query and apply the Stream UDF use `query.apply()`, passing the
UDF module name, function and arguments (if any):

```js

client.apply('example', 'count', function (error, result) {
  if (!error) {
    console.log(result)
  }
})
```
