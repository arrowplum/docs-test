---
title: Query Records
description: The Aerospike Rust client provides an API to query the database using secondary indexes.
---

Use the Aerospike Rust client APIs to query the database using secondary
indexes.

Read **[Manage Indexes](/docs/client/rust/usage/query/sindex.html)** for
creating secondary indexes.

### Defining the Query

Use `query::Statement` to define a query.

First, create an instance of `Statement` by specifying the namesapce _foo_ and
set _bar_ to query. Optionally, a list of bin names can also be specified to
restrict, which record bins will be returned by the query.

```rust
use aerospike::Statement;

let mut stmt = Statement::new("foo", "bar", Some(&vec!["name", "age"]));
```

{{#note}}
Namespace is required when querying using secondary indexes; however, sets are
optional, depending on the secondary index.
{{/note}}

#### Applying Filters

To query a secondary index, specify a filter on the query.  It appears that
multiple filters are allowed, but the server currently restricts queries to a
single filter.

Filters are creating using the macros defined in the `aerospike.query` module.
The following filters are currently supported:

- `as_eq!` &mdash; Create equality filter for queries; supports integer and
  string values.
- `as_range!` &mdash; Create range filter for queries; supports integer values.
- `as_contains!` &mdash; Create contains number filter for queries on a
  collection index.
- `as_contains_range!` &mdash; Create contains range filter for queries on a
  collection index.
- `as_within_region!` &mdash; Create geospatial "points within region" filter
  for queries.
- `as_within_radius!` &mdash; Create geospatial "points within radius" filter
  for queries.
- `as_regions_containing_point!` &mdash; Create geospatial "regions containing
  point" filter for queries.

This example uses a numeric index on bin _baz_ in namespace _foo_ within set _bar_ to find all records using a filter with the range 0 to 100 inclusive:

```rust
stmt.add_filter(as_range!("baz", 0, 100));
```

{{#note}}
Filters are optional. If not specified, the scan runs over the entire namespace and/or set.
{{/note}}

### Executing the Query

To execute the query, invoke `Client.query`:

```rust
    pub fn query(&self,
                         policy: &QueryPolicy,
                         statement: Statement)
                         -> Result<Arc<Recordset>> {
```

Where,

- `policy` &mdash; Query behavior definition.
- `statement` &mdash; Query to execute.

`RecordSet` allows you to iterate over the results of the query.

To execute the query and iterate over the results:

```rust
match client.query(&QueryPolicy::default(), stmt) {
    Ok(records) => {
        for record in &*records {
            // .. process record
        }
    },
    Err(err) => println!("Error fetching record: {}", err),
}
```
