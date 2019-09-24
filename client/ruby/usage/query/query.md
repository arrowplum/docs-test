---
title: Query Records
description: Use the Aerospike Ruby client to query the database using secondary indexes.
---

Use the Aerospike Ruby client to query the database using secondary indexes.

Read the following topics before continuing:

- [Manage Indexes](/docs/client/ruby/usage/query/sindex.html)

### Defining a Query

Use the Aerospike Ruby client library `Statement` class to define a query against the database.

This example creatis an instance of `Statement` called *stmt*:

```ruby
stmt = Statement.new("ns", "set", ["bin1", "bin2"])
```

{{#note}}
The namespace is required when querying using secondary indexes; however, the set may be optional, depending on the secondary index. You can pass an empty string to omit it.
{{/note}}

#### Filters

To query a secondary index, specify the filters on the query:

```ruby
stmt.filters << filter1
```

Filters are created using predefined initializers. The following are the available filter types:

- `Filter.Equal(binName, value)` &mdash; Filter record bin *binName* containing the specified value.
- `Filter.Range(binName, begin, end)` &mdash; Filter record bin *binName* containing a value in the specified range.

With a numeric index on bin *baz* in the namespace *ns* and set *set*, add a filter to find all records containing bin *baz* with values between 0 and 100:

```ruby
stmt.filters << Filter.Range("baz", 0, 100))
```

{{#note}}
The filter is optional. If a filter is not specified, the scan is performed over the specified set and namespace.
{{/note}}

#### Geospatial Filters

For record bins creating geospatial data in the form of GeoJSON objects, a geospatial index can be created, that supports querying the data using points-within-region and region-contains-point filters:

- `Filter.geoWithinGeoJSONRegion(binName, region)` &mdash; Filter record bin *binName* containing a GeoJSON object representing a point within the specified GeoJSON region.
- `Filter.geoWithinRadius(binName, lon, lat, radius)` &mdash; Filter record bin *binName* containing a GeoJSON object representing a point within the specified radius in meter from the lat/lon.
- `Filter.geoContainsGeoJSONPoint(binName, point)` &mdash; Filter record bin *binName* containing a GeoJSON object representing a region (e.g. polygon) which contains the specified GeoJSON point.
- `Filter.geoContainsPoint(binName, lon, lat)` &mdash; Filter record bin *binName* containing a GeoJSON object representing a region (e.g. polygon) which contains the specified lat/lon.

{{#note}}
Geospatial indexes require server version 3.7.0 or newer.
{{/note}}

*Example:*

With a geospatial index on bin `loc`, find all records where the bin `loc` represents a GeoJSON region that contains a point with the coordinates (103.91146, 1.30838).

```ruby
poi = GeoJSON.new({ type: "Point", coordinates: [103.91146, 1.30838] })
stmt.filters << Filter.geoContainsGeoJSONPoint("loc", poi)
```

### Executing Query

To execute the query using `#query`:

```ruby
client.query(stmt)
```

Where `stmt` is the query to execute.

The return value is an instance of `Recordset` that provides the ability to iterate over the results of the query.

To execute the query and iterate over the results:

```ruby
rs = client.query(stmt)

rs.each do |rec|
  # process the record
end
```

