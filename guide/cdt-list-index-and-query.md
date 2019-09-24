---
title: List Indexing and Querying
description: Defining secondary indexes and running queries on Aerospike list values
---

Aerospike allows users to create a [secondary index](/docs/guide/query.html#secondary-index-key-data-type)
on bins whose data type is a list.

## Indexing on List Elements

- Similar to basic indexing, the indexable list element data types are numeric, string, and GeoJSON.
- When creating index, users need to specify explicitly that list bins should be indexed, and what data type to index on.
- When querying, users need to specify that the query should be applied on a CDT data type.
- Similar to basic [querying](/docs/guide/query.html), equality, range (for numeric and string data type), points-within-region, region-containing-points (for GeoJSON data type) are supported.

In this example we use the [_aql_](/docs/tools/aql) tool to create two indexes with source type `LIST`, one for values with a numeric data types, one for values with a string data types.

```sql
CREATE LIST INDEX foo_list_ints ON test.demo (foo) NUMERIC
CREATE LIST INDEX foo_list_strs ON test.demo (foo) STRING
```

Elements of the indexed list are type checked, so a record whose `foo` bin
contains `[ 1, "2", 3, [4], 5 ]` results in the following indexing:

| Index On | Key Type | Index Type | Eligible Secondary Index Key |
| --- | --- | --- | --- | --- | --- |
| foo | string |	LIST	| "2" |
| foo | numeric | LIST	| 1, 3, 5 |

### List Index Queries

The following example uses the aql tool to query an indexed list.

```bash
aql> INSERT INTO test.demo (PK, username, emails) VALUES ("u1", "Bob Roberts", LIST('["bob.roberts@gmail.com", "bob@yahoo.com"]'))
OK, 1 record affected.

aql> INSERT INTO test.demo (PK, username, emails) VALUES ("u2", "rocketbob", LIST('["bigb@gmail.com", "bob@yahoo.com"]'))
OK, 1 record affected.

aql> INSERT INTO test.demo (PK, username, emails) VALUES ("u3", "samunwise", "pppreciousss@gmail.com")
OK, 1 record affected.

aql> CREATE LIST INDEX email_idx ON test.demo (emails) STRING
OK, 1 index added.

aql> show indexes
+--------+----------+-----------+--------+-------+-------------+----------+----------+
| ns     | bin      | indextype | set    | state | indexname   | path     | type     |
+--------+----------+-----------+--------+-------+-------------+----------+----------+
| "test" | "emails" | "LIST"    | "demo" | "RW"  | "email_idx" | "emails" | "STRING" |
+--------+----------+-----------+--------+-------+-------------+----------+----------+
[127.0.0.1:3000] 1 row in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN LIST WHERE emails = "bigb@gmail.com"
+-------------+
| username    |
+-------------+
| "rocketbob" |
+-------------+
1 row in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN LIST WHERE emails = "bob@yahoo.com"
+---------------+
| username      |
+---------------+
| "Bob Roberts" |
| "rocketbob"   |
+---------------+
2 rows in set (0.001 secs)

OK

aql> SELECT username FROM test.demo IN LIST WHERE emails = "pppreciousss@gmail.com"
0 rows in set (0.001 secs)

OK

aql> SELECT username FROM test.demo WHERE emails = "pppreciousss@gmail.com"
0 rows in set (0.001 secs)

Error: (201) AEROSPIKE_ERR_INDEX_NOT_FOUND

```

## Known Limitations

- List indexing is only applied to top level elements. No indexing (yet) on a nested list.
- When using range queries on lists, records can be returned multiple times if the list contains multiple values that falls within the range.

