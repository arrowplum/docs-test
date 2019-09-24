---
title: Map Indexing and Querying
description: Defining secondary indexes and running queries on Aerospike map keys and values
---

Aerospike allows users to create a [secondary index](/docs/guide/query.html#secondary-index-key-data-type)
on bins whose data type is a map. Either map key or map value can be indexed.

## Indexing on Map Elements

- Similar to basic indexing, the indexable map data types are numeric, string, and GeoJSON.
- When creating index, users need to specify explicitly that map bins should be indexed, and what data type to index on. 
- When querying, user need to specify that query should be applied on a CDT data type.
- Similar to basic querying, equality, range (for numeric and string data type), points-within-region and region-containing-points (for geoJSON data type) are supported.

In this example we use the [_aql_](/docs/tools/aql) tool to create two indexes, one with source type `MAPKEYS` and a string data type, one with source type `MAPVALUES` with a numeric data type:

```sql
CREATE MAPKEYS INDEX foo_mapkey_idx ON test.demo (foo) STRING
CREATE MAPVALUES INDEX foo_mapval_idx ON test.demo (foo) NUMERIC
```

Elements of the indexed list are type checked, so a record whose `foo` bin
contains `{ a:1, b:"2", c:3, d:[4], e:5, 666:"zzz" }` will results in the following indexing:

| Index On | Key Type | Index Type | Eligible Secondary Index Key |
| --- | --- | --- | --- | --- | --- |
| foo | string | MAPKEYS | a, b, c, d, e |
| foo | numeric| MAPVALUES | 1, 3, 5 |

## Map Index Queries

This example illustrates use of map indexes using aql scripts. The example creates index over bin `shopping_carts` which is map of products and state from which it was ordered and queries the record based product name or state.

```bash
aql> CREATE MAPKEYS INDEX foo_mapkey_idx ON test.demo (foo) STRING
OK, 1 index added.

aql> CREATE MAPVALUES INDEX foo_mapval_idx ON test.demo (foo) NUMERIC
OK, 1 index added.

aql> show indexes
+--------+-------+-------------+--------+-------+------------------+-------+-----------+
| ns     | bin   | indextype   | set    | state | indexname        | path  | type      |
+--------+-------+-------------+--------+-------+------------------+-------+-----------+
| "test" | "foo" | "MAPKEYS"   | "demo" | "RW"  | "foo_mapkey_idx" | "foo" | "STRING"  |
| "test" | "foo" | "MAPVALUES" | "demo" | "RW"  | "foo_mapval_idx" | "foo" | "NUMERIC" |
+--------+-------+-------------+--------+-------+------------------+-------+-----------+
[127.0.0.1:3000] 2 rows in set (0.001 secs)

OK

aql> INSERT INTO test.demo (PK, username, foo) VALUES ("u1", "Bob Roberts", JSON('{"a":1, "b":"2", "c":3, "d":[4]}'))
OK, 1 record affected.

aql> INSERT INTO test.demo (PK, username, foo) VALUES ("u2", "rocketbob", MAP('{"c":3, "e":5}'))
OK, 1 record affected.

aql> INSERT INTO test.demo (PK, username, foo) VALUES ("u3", "samunwise", JSON('{"x":{"z":26}, "y":"yyy"}'))
OK, 1 record affected.

aql> SELECT username FROM test.demo IN MAPVALUES WHERE foo = 2
0 rows in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN MAPVALUES WHERE foo = "2"
0 rows in set (0.000 secs)

Error: (201) AEROSPIKE_ERR_INDEX_NOT_FOUND

aql> SELECT username FROM test.demo IN MAPVALUES WHERE foo = 1
+---------------+
| username      |
+---------------+
| "Bob Roberts" |
+---------------+
1 row in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN MAPVALUES WHERE foo = 3
+---------------+
| username      |
+---------------+
| "Bob Roberts" |
| "rocketbob"   |
+---------------+
2 rows in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN MAPVALUES WHERE foo BETWEEN 1 AND 5
+---------------+
| username      |
+---------------+
| "Bob Roberts" |
| "Bob Roberts" |
| "rocketbob"   |
| "rocketbob"   |
+---------------+
4 rows in set (0.001 secs)

aql> SELECT username FROM test.demo IN MAPKEYS WHERE foo = "y"
+-------------+
| username    |
+-------------+
| "samunwise" |
+-------------+
1 row in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN MAPKEYS WHERE foo = "x"
+-------------+
| username    |
+-------------+
| "samunwise" |
+-------------+
1 row in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN MAPKEYS WHERE foo = "c"
+---------------+
| username      |
+---------------+
| "Bob Roberts" |
| "rocketbob"   |
+---------------+
2 rows in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN MAPKEYS WHERE foo = "z"
0 rows in set (0.001 secs)

OK
```

{{#note}}
The aql tool doesn't allow for numeric map keys, so those aren't used in this
example. However, that is not a limitation of the Aerospike server, or of the
Aerospike clients for Java, Python, Go, etc.
{{/note}}

## Known Limitations

- Map indexing is only applied to top level elements. No indexing (yet) on a nested map.
- When using range queries on maps, records can be returned multiple times if the map contains multiple values that falls within the range.
