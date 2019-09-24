---
title: Aggregate Records
description: Use the Aerospike Java client APIs to aggregate records to process query results.
---

There are various forms of processing results of a [secondary index query](/docs/client/java/usage/query/sindex.html). A common form is aggregation, where you apply a function on the entire results set of a query.

Many developers use SQL to define aggregation queries against a database. For example, the following SQL statement counts rows from the database:

```sql
SELECT count(*)
FROM test.demo 
WHERE d = 50
```

This counts the number of records in _test.demo_ that contain a column _d_ equal _50_. In Aerospike you can use UDFs and queries to do this.

Read these sections before continuing:

- [Connecting](/docs/client/java/usage/connect) &mdash; Establishing cluster connections.
- [Manage Indexes](/docs/client/java/usage/query/sindex.html) &mdash; Creating secondary indexes.
- [Register UDF](/docs/client/java/usage/udf/register.html) &mdash; Registering UDFs.
- [Query Records](/docs/client/java/usage/query/query.html) &mdash; Creating and executing queries.

### Defining the Query

Read [Query Records](/docs/client/java/usage/query/query.html) to learn how define a query:

```java
Statement stmt = new Statement()
stmt.setNamespace("foo");
stmt.setSetName("bar");
stmt.setFilters( Filter.range("baz", 0,100) );
stmt.setBinNames("baz");
```

This query returns all records in the namespace _foo_ and sets _bar_, where the bin _baz_ contains a value between _0_ and _100_.

### Defining the Stream UDF

To count the number of records that satisfy the query, define a UDF to count all returned records. When a query executes, it produces a stream of results. [Query Records](/docs/client/java/usage/query/query.html) explains that the stream contains records that you can iterate using the client API. However, Aerospike provides the ability to process the stream of results using a **Stream UDF**. Stream UDFs allow a stream to be augmented with operations that process the data flowing through the stream.

This example defines a Stream UDF _count_, which is in the _example.lua_ module:

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

`count()` is applied to a stream (in this case, the stream of results from the query). We add to the stream the operations we want to perform on the results:

- `map` &mdash; Maps a value from the stream to another value. In the example, mapping is defined as the function `one()`, which maps a record to the value 1.
- `reduce` &mdash; Reduces the values from the stream into a single value. In the example, reduction is performed by adding two values from the stream, which happen to be 1s returned from the `map` function.

The end result is a stream that contains a single value: the _count_ or more technically, the sum of 1 for each record in the result set.

### Registering the UDF

The UDF module must be registered with the cluster before you can use a Stream UDF. See [Registering UDF](/docs/client/java/usage/udf/register.html). The following example registers the _example.lua_ UDF module:

```java
client.register(null, "udf/example.lua", "example.lua", Language.LUA);
```

### Configuring the UDF Search Path

For client-side Stream UDF processing, you must point the client to the local location of the UDF module. Define the UDF module location using  `com.aerospike.client.lua.LuaConfig`:

```java
LuaConfig.SourceDirectory = "udf";
```

### Executing the Query

To execute the query and process the results, use `AerospikeClient.queryAggregate()`:

```java
public class AerospikeClient {
    public final ResultSet queryAggregate(QueryPolicy policy,
        Statement statement,
        String packageName,
        String functionName,
        Value... functionArgs
    ) throws AerospikeException
}
```

To call `example.count()`: 

```java
ResultSet rs = client.queryAggregate(null, stmt, "example", "count");
```

### Processing the Results

The query produces a `com.aerospike.client.query.ResultSet`, which allows you to iterate the results of the aggregation:

```java
if (rs.next()) {
    Object result = rs.getObject();
    System.out.println("Count = " + result);
}
```

### Cleaning Up Resources

After the query results complete processing, close the `ResultSet`:

```java
rs.close();
```
