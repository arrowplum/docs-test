---
title: Aggregate Records
description: Use the Aerospike Python client to define aggregation queries for the Aerospike database.
---

There are various forms of processing results of a [secondary index query](/docs/client/python/usage/udf/query.html). A common form is aggregation, where you apply a function on the entire results set of a query.

Developers familiar with SQL aggregation queries will recognize the following SQL statement for counting rows from the database:

```sql
SELECT count(*)
FROM test.demo 
WHERE d = 50
```

The statement counts the number of records in _test.demo_, containing column _d_ equal to *50*. In Aerospike you use a query and UDFs to perform this operation.

Read the following topics before continuing:

- [Connecting](/docs/client/python/usage/connect) &mdash; Establish a cluster connection.
- [Query Records](/docs/client/python/usage/udf/query.html) &mdash; Create and execute queries.

### Defining the Query

Construct a query as defined in [Query](/docs/client/python/usage/udf/query.html). 

This example queries namespace *test* and set *demo*:

```python
from aerospike import predicates as p

query = client.query( 'test', 'demo' )
query.select( 'name', 'age' )
query.where( p.between('age', 14, 25) )
```

### Applying a Stream UDF

A **Stream UDF** defines operations to apply to a stream of query results.

{{#note}}
UDF modules must register with the database. 
{{/note}}

In this example, the `mymodule` UDF module defines `count()`, which is a Stream UDF. In `count()`, each record transforms to `1` (one) and adds each `1` to produce the count of records in the query.

```lua
local function one(rec)
    return 1
end
 
local function add(a, b)
    return a + b
end
 
function count(stream)
    return stream : map(one) : reduce(add);
end
```

To invoke a UDF on query results, use `apply()` on the `Query` class instance:

```python
query.apply('mymodule', 'count')
```

### Reading Results

To execute the query and read query results, use `foreach()` in the `Query` class instance. `foreach()` accepts a callback function called for each result read from the query. The callback function must accept a single argument as the values read from the query.

These examples execute the query and read results.

To print records as they are read:

```python
def print_result(value):
    print(value)
```

To execute the query and call `print_result` for each result:

```python
query.foreach(print_result)
```

