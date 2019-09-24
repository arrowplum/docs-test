---
title: Query Records
description: Use the Aerospike Nodejs client to query the database using secondary indexes.
---

Use the Aerospike Node.js client to query the database using secondary indexes.

Read the following topics before continuing:

- **[Connecting](/docs/client/nodejs/usage/connect)** &mdash; Establish a cluster connection.
- **[Manage Indexes](/docs/client/nodejs/usage/query/sindex.html)** &mdash; Create a secondary index.
- **[Register UDF](/docs/client/nodejs/usage/udf/manage.html)** &mdash; Register a user-defined function.

### Defining the Query

The `Query` object in Aerospike Nodejs Client defines a query against the database.

To create a `Query` instance use the `Client#query` method and specify the namespace and set to query:

```js
var query = client.query('test', 'demo')
```

{{#note}}
The namespace is required when querying using secondary indexes; however, the
set may be optional, depending on the secondary index.
{{/note}}

#### Filters

You can limit the query results by supplying a filter predicate. The query will
apply the predicate against all records in the namespace/set and only return
the matching records. Filter predicates are created using the helper functions
in the `aerospike.filter` submodule. To add the predicate to the query use the
`Query#where` method.

These are the currently available filter predicates:

- `equal` &mdash; String/integer equality. The predicate
  matches records with a bin that matches the specified string or integer
  value.
- `range` &mdash; Integer range filter. The filter matches
  records with a bin value in the given integer range.
- `contains` &mdash; Filter for list/map
  membership. The filter matches records with a bin that has a list or map
  value that contains the given string or integer.
- `geoWithinGeoJSONRegion` &mdash; Geospatial filter that matches points within
  a given GeoJSON region.
- `geoWithinRadius` &mdash; Geospatial filter that matches points within a
  radius from a given point.
- `geoContainsGeoJSONPoint` &mdash; Geospatial filter that matches GeoJSON
  regions that contain a given point.
- `geoContainsPoint` &mdash; Geospatial filter that matches regions that
  contain a given lng/lat coordinate.

This example adds a filter to find all records with bin _baz_ containing integer values between 0 and 100:

```js
query.where(Aerospike.filter.range('baz', 0, 100))
```

{{#note}}
Filters are optional. If the filter is not specified, all of the records in the
namespace/set will be returned. Alternatively, a [Scan](/docs/client/nodejs/usage/scan) operation can
be performed, which allows more fine grained control over execution priority,
concurrency, etc.
{{/note}}

#### Projecting Bins

A query can specify an array of bins to read from the database.

To project a single bin _baz_:

```js
query.select('baz')
```

### Executing the Query

To execute the query and iterate through the result set, invoke `Query#foreach`:

```js
var stream = query.foreach()
```

The return value is a stream, which emits `data` event for every record that the query returns.

To execute the query and process the results:

```java
var stream = query.foreach()
stream.on('data', function (record) {
  //process record
})
stream.on('error', function (error) {
  //handle error
})
stream.on('end', function () {
  //signal the end of query result
})
```
