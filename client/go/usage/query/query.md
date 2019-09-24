---
title: Query Records
description: Use the Aerospike Go client to query the database using secondary indexes.
---

As well as querying using the primary index using the [KVS API](/docs/client/go/usage/kvs/read.html), you can query the database using secondary indexes.

See **[Manage Indexes](/docs/client/go/usage/query/sindex.html)** to create secondary indexes.

### Defining the Query

The Aerospike Go client library uses `Statement` to define a query against the database.

To create an instance of `Statement` called `stmt`:

```go
stmt := as.NewStatement("foo", "bar", "bin1", "bin2")
```

{{#note}}
The namespace is required when querying using secondary indexes; however, the set may be optional, depending on the secondary index. You can pass an empty string to omit it.
{{/note}}

#### Applying Filters

You create filters using predefined initializers. The following filters are available:

- `NewEqualFilter(binName, value)` – Filter record with bin _binName_ that contains the specified value.
- `NewRangeFilter(binName, begin, end)` – Filter record with bin _binName_ that contains the value in the specified range.


To query a secondary index, specify filters for the query:

```go
func (stmt *Statement) Addfilter(filter *Filter) error
```

With a numeric index on bin _baz_ in namespace _foo_ within set _bar_, to add a filter to find all records that contain the bin _baz_ with values between 0 and 0:

```go
stmt.Addfilter(as.NewRangeFilter("baz", 0, 100))
```

{{#note}}
Filters are optional. If not specified, the scan is performed over the specified set and namespace.
{{/note}}

### Executing Query

To execute the query using `Client.Query()`:

```go
func (clnt *Client) Query(policy *QueryPolicy,
    statement *Statement) (*Recordset, error)
```

Where,

- `policy` &mdash; Set query behavior (default = `null`).
- `statement` &mdash; Query to execute.

The return value is a `*Recordset`, which provides the ability to iterate over the results of the query.

To execute the query and iterate over the results:

```go
rs, err := client.Query(nil, stmt)
// deal with error

for res := range rs.Results() {
  if res.Err != nil {
    // handle error here
    // if you want to exit, cancel the recordset to release the resources
  } else {
    // process record here
    fmt.Println(res.Record.Bins)
  }
}
```

