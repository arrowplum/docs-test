---
title: aql – Querying Records
description: See examples of how aql can be used for querying records such as scanning records in a set, querying project bins from records, filtering on indexed bins and aggregating on query results.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: aql
    url: /docs/v3/aql.html
---

### Scan All Records in a Set

The following is a query that scans all records from a specific namespace and set:

```sql
SELECT * FROM <ns>[.<set>]
```
Where

- `<ns>` is the namespace for the records to be queried.
- `<set>` is the set name for the record to be queried.

Example:

```sql
aql> SELECT * FROM users.profiles
+---------------------------+-----+--------+
| name                      | age | gender |
+---------------------------+-----+--------+
| "Bob White"               | 22  | "M"    |
| "Annie Black"             | 28  | "F"    |
| "Sally Green"             | 19  | "F"    |
| "Ricky Brown"             | 20  | "M"    |
| "Tammy Argent"            | 22  | "F"    |
+---------------------------+-----+--------+
5 rows in set (0.000 secs)
```

{{#note}}
**Note:** Starting Tools version 3.8.2, `SELECT *` will also print the primary key of a record n a column 'PK' if it was sent to the server when the record was written.
{{/note}}

### Project Specific Bins

The following is the syntax for a query to project bins from all records in a specific namespace and set.

```sql
SELECT <bin>[, <bin>[, ...]] FROM <ns>[.<set>]
```

Where

- `<ns>` is the namespace for the records to be queried.
- `<set>` is the set name for the record to be queried.
- `<bin>` are one or more bins to project from the records.

Example:

```sql
aql> SELECT name, age FROM users.profiles
+---------------------------+-----+
| name                      | age |
+---------------------------+-----+
| "Bob White"               | 22  |
| "Annie Black"             | 28  |
| "Sally Green"             | 19  |
| "Ricky Brown"             | 20  |
| "Tammy Argent"            | 22  |
+---------------------------+-----+
5 rows in set (0.000 secs)
```

### Filtering on Indexed Bins

The following is the syntax for a filtering all records in a specific namespace and set based on a predicate:

```sql
SELECT <bin>[, <bin>[, ...]] FROM <ns>[.<set>] [IN indextype] WHERE [<predicate>]
```

the index type specifier 

```sql
IN indextype
```

is required in case the index being queried is on [LIST](/docs/guide/cdt-list.html#list-index), [MAPKEYS](/docs/guide/cdt-map.html#map-indexes), [MAPVALUES](/docs/guide/cdt-map.html#map-indexes). If not specified defaults on index on basic index on bin itself.

The `<predicate>` must be one of the supported predicates for the index. For NUMERIC indexes, either a range predicate or equality predicate can be applied. For STRING indexes, only equality predicates are supported. Starting tools version 3.8.2 - for GeoJSON, both equality and range predicates are supported by using 'CONTAINS' and 'WITHIN'.


A predicate can be in the following forms depending on the datatype:

```sql
# For all datatypes, this states that the bin `<bin>` must fall with then range between `<lower>` and `<upper>` (inclusive).
<bin> BETWEEN <lower> AND <upper>
# For GeoJSON, this returns value containing specific location points.
<bin> CONTAINS <GeoJSONPoint>
# For GeoJSON, this returns values within a specified range of points.
<bin> WITHIN <GeoJSONPolygon>
```

Example of a GeoJSON range query:

```sql
SELECT * FROM test.demo WHERE gj CONTAINS CAST('{"type": "Point", "coordinates": [0.0, 0.0]}' AS GEOJSON)
```

An equality predicate is in the form which states that the bin `<bin>` must equal `<value>`.

```sql
<bin> = <value>
```

Or, if attempting to get a record for a specific primary key.

```sql
PK = <key>
```

In the following example, we assume we have an index on the age bin, and want to query all people between the age of 20 and 29

```sql
aql> SELECT name, age FROM users.profiles WHERE age BETWEEN 20 AND 29
+---------------------------+-----+
| name                      | age |
+---------------------------+-----+
| "Bob White"               | 22  |
| "Annie Black"             | 28  |
| "Ricky Brown"             | 20  |
| "Tammy Argent"            | 22  |
+---------------------------+-----+
4 rows in set (0.000 secs)
```

Assuming a secondary index for the above would have been created using:

```sql
aql> CREATE INDEX user_age_idx ON users.profiles (age) NUMERIC
```

### Aggregating on Query or Scan Results

The following is the command to perform an aggregation on secondary index results.

```sql
AGGREGATE <module>.<function>([<arg>[,...]]) ON <ns>[.<set>] [IN indextype] WHERE <predicate>
```

Remove "WHERE predicate" to run aggregation on scan results as below.

```sql
AGGREGATE <module>.<function>([<arg>[,...]]) ON <ns>[.<set>]
```

This specifies a Stream UDF to execute against the results of a secondary index query.

Where 

- `<module>` is the UDF module name.
- `<function>` is the Stream UDF function name.
- `<arg>` are the arguments for the UDF function. Multiple arguments can be specified.
- `<predicate>` is the same predicate used for querying. 

In the following example, the query will have an aggregation stream applied to it called "avg_age". "avg_age" has the appropriate sub-functions required to define the different stages of the stream (function "female", "name_age", and "eldest"). This set of fuctions are included in the UDF module named "profile_aggregator":

```sql
aql> AGGREGATE profile_aggregator.avg_age() ON users.profiles WHERE age BETWEEN 20 and 29
+--------------------------------------+
+ avg_age                        |
+--------------------------------------+
+ { "name": "Annie Black", "age": 28 } |
+--------------------------------------+
```

Assuming the follwoing UDF module named "profile_aggregator" was registered containing an "avg_age" function like:

```lua
function avg_age(stream)
  
  local function female(rec)
    return rec.gender == "F"
  end
 
  local function name_age(rec)
    return map{ name=rec.name, age=rec.age }
  end
   
  local function eldest(p1, p2)
    if p1.age > p2.age then 
      return p1
    else
      return p2
    end
  end
 
  return stream : filter(female) : map(name_age) : reduce(eldest)
end
```

The stream operations defined will:

1. filters female users
2. converts each profile record into a map containing only a name and age
3. reduces each profile to eldest female

For additional information, please see:
- [Developing Stream UDF's](/docs/udf/developing_stream_udfs.html)
- [Aggregation Feature Guide](/docs/guide/aggregation.html)
 
