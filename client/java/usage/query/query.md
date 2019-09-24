---
title: Query Records
description: The Aerospike Java client provides an API to query the database using secondary indexes.
---

Use the Aerospike Java client APIs to query the database using secondary indexes.

Read **[Manage Indexes](/docs/client/java/usage/query/sindex.html)** for creating secondary indexes.

### Defining the Query

Use `com.aerospike.client.query.Statement` to define a query.

First, create an instance of `Statement` called `stmt`:

```java
Statement stmt = new Statement()
```

To query a specific namespace or set, use the following methods to set the values. This example queries the namespace _foo_ in set _bar_:

```java
stmt.setNamespace("foo");
stmt.setSetName("bar");
```

{{#note}}
Namespace is required when querying using secondary indexes; however, sets are optional, depending on the secondary index.
{{/note}}

#### Applying Filters

To query a secondary index, specify a filter on the query.  It appears that multiple filters are allowed, but the server currently restricts queries to a single filter:


```java
public final class Statement {
    void setFilters(Filter... filters);
}
```

Filters are creating using the static methods in `com.aerospike.client.query.Filter`. The following are available filters:

- `Filter.equal(name, value)` &mdash; Filters the record with bin _name_ with a specified value (integer or string).
- `Filter.range(name, begin, end)` &mdash; Filters record with bin _name_ with a value in the specified range (integer only).

This example uses a numeric index on bin _baz_ in namespace _foo_ within set _bar_ to find all records using a filter with the range 0 to 100 inclusive:

```java
stmt.setFilters(Filter.range("baz", 0, 100));
```

{{#note}}
Filters are optional. If not specified, the scan runs over the entire namespace and/or set.
{{/note}}

#### Projecting Bins

A query can specify bins to read:

```java
public final class Statement {
    void setBinNames(String... binNames);
}
```

To project the bin _baz_:

```java
stmt.setBinNames("baz");
```

### Executing the Query

To execute the query, invoke `AerospikeClient.query`:


```java
public class com.aerospike.client.AerospikeClient {
    public final RecordSet query(
        QueryPolicy policy,
        Statement statement
    )   throws AerospikeException;
}
```

Where,

- `policy` &mdash; Query behavior definition. Default = `null`.
- `statement` &mdash; Query to execute.

`RecordSet` always returns, which allows you to iterate over the results of the query.

To execute the query and iterate over the results:

```java
RecordSet rs = client.query(null, stmt);

try {
    while (rs.next()) {
        Key key = rs.getKey();
        Record record = rs.getRecord();
    }
}
finally {
    rs.close();
}
```
