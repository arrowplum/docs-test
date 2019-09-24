---
title: aql – Index Management
description: Learn how aql can be used to manage indexes, such as creating, listing, dropping and repairing an index.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: aql
    url: /docs/v3/aql.html
---

### Creating an Index

The following is the command to create a new secondary index:

```sql
CREATE [indextype] INDEX <index> ON <ns>[.<set>] (<bin>) <type>
```

This will create an index named `<index>` on bin `<bin>` of type `<type>` in the namespace `<ns>` and set `<set>`.

Where

- `<indextype>` is the type of index (LIST, MAPKEYS, MAPVALUES). The value is optional if not specified defaults to basic index on the bin
- `<index>` is the name of the index to be created. The index name must be unique within a namespace.  The index name should not exceed 20 characters in length. The index name should not include colon (:) or semicolon (;). 
- `<ns>` is the namespace to which the index will belong, but is also the namespace containing the record to be indexed. 
- `<set>` is the set, which will contain the records to be indexed. The `<set>` is optional, and if not specified, will index records not belonging to a set.
- `<bin>` is the name of the bin to be indexed. 
- `<type>` is the type of the value stored in the <bin> must be either: `NUMERIC` or `STRING` or `GEO2DSPHERE`.

Additional indexing rules:

- Any record containing bin `<bin>`, but does not match the specified `<type>` will not be indexed.
- Any record that does not contain bin `<bin>` will not be indexed.

The secondary index will be built in the background, and will not be query-able until it is completely built.  If you attempt to query on an index that is not yet complete, you will receive an Index not active error.

The following is an example of creating a "numeric" index named "numindex" on bin "binB" of records in "test" namespace and "testset" set.

```sql
aql> CREATE INDEX numindex ON test.testset (binB) NUMERIC
```
The following is an example of creating a "string" list type index named "strindex" on bin "binB" of records in "test" namespace and "testset" set.

```sql
aql> CREATE LIST INDEX strindex ON test.testset (binB) STRING
```

The following is an example of creating a "geospatial" index named "geoindex" on bin "geobin" or records in "test" namespace and "testset" set.

```sql
aql> CREATE INDEX geoindex ON test.testset (geobin) GEO2DSPHERE
```

### Listing Indexes

The following is the command to list all the indexes within a cluster:

```sql
SHOW INDEXES [<ns>]
```

The `<ns>` parameter, will only list indexes in the specified namespace.

The following is an example of listing indexes in "test" namespace:

```sql
aql> show indexes test
+--------+--------+-----------+-----------+-------+------------+--------+------------+-----------+
| ns     |  bin   | indextype |    set    | state | indexname  |  path  | sync_state |   type    |
+--------+--------+-----------+-----------+-------+------------+--------+------------+-----------+
| "test" | "binB" |  "NONE"   | "testset" | "RW"  | "numindex" | "binB" | "synced"   | "NUMERIC" |
| "test" | "binC" |  "LIST"   | "testset" | "RW"  | "strindex" | "binC" | "synced"   | "STRING"  |
+--------+--------+-----------+----------+-------+------------+---------+------------+-----------+
2 rows in set (0.001 secs)
```

Details on the `sync_state` and state are explained below in Index Sync State and Index State sections.

### Dropping an Index

The following is the command to drop (remove) a secondary index from a namespace:

```sql
DROP INDEX <ns> <index>
```

This will atomically drop (remove) the index named `<index>` in the `<ns>` namespace across the entire cluster.

Where
- `<ns>` is the namespace containing the index to be dropped.
- `<index>` is the name of the index to be dropped.

The following is an example of dropping the "numindex" index from "test" namespace:

```sql
aql> drop index test numindex
```

### Index Sync State

Deprecated in version 3.14 and above.

Each secondary index has a value called sync_state, which specifies whether the secondary index is in sync with the primary index.

Value | Description
--- | ---
`synced` | The secondary index is in sync with the primary index.
`need_sync` | The secondary index may not be in sync with the primary index.

{{#note}}If sync_state=need_sync and state=RW, then the secondary index may need to be repaired.{{/note}}

### Index State

Each secondary index has a value called state, which specifies 

Value | Description
--- | ---
WO | Secondary index in Write Only Mode. Normal aerospike put will update secondary index but queries cannot be performed.
RW | Secondary index in Read Write Mode. Queries will work in this mode.

{{#info}}A secondary index should be in RW state on all the nodes before query can use it.{{/info}}

### Repairing an Index

{{#note}}
Deprecated in versions 3.14 and above.
{{/note}}


When an index has `sync_state=need_sync` and `state=RW`, then it may need to be repaired.

aql does not provide the ability to repair indexes. For the time being, you will need to use Aerospike Info (asinfo) to execute a repair.

```bash
$ asinfo -v "sindex-repair:ns=test;indexname=ind_name;set=set_name;"
```
 
