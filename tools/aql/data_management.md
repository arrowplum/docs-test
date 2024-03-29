---
title: aql – Data Management
description: Learn how to use aql for data management. Use aql to list namespaces, sets, bins, and record details.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: aql
    url: /docs/v3/aql.html
---

### List Namespaces

The following command lists the namespaces:

```sql
SHOW NAMESPACES
```

Example:

```sql
aql> show namespaces
+------------+
| namespaces |
+------------+
| "test"     |
| "bar"      |
+------------+
2 rows in set (0.001 secs)
```

### List Sets

The following command lists all sets:

```sql
SHOW SETS
```

Example:

```sql
aql> show sets
+---------+-----------+-----------+------------+-----------+---------+
| ns_name | set_name  | n_objects | stop-write | evict-hwm | delete  |
+---------+-----------+-----------+------------+-----------+---------+
| "test"  | "newtest" | 11        | 0          | 0         | "false" |
| "test"  | "users"   | 1         | 0          | 0         | "false" |
+---------+-----------+-----------+------------+-----------+---------+
2 rows in set (0.001 secs)
```

### List Bins

The following command lists all bins:

```sql
SHOW BINS
```

Example:

```sql
aql> show bins
+-----------+-------+-------+-----------+
| namespace | count | quota | bin       |
+-----------+-------+-------+-----------+
| "test"    | 5     | 32768 | "b"       |
| "test"    | 5     | 32768 | "c"       |
| "test"    | 5     | 32768 | "a"       |
| "test"    | 5     | 32768 | "SUCCESS" |
| "test"    | 5     | 32768 | "name"    |
+-----------+-------+-------+-----------+
5 rows in set (0.000 secs)
```

### List Record Details

Note: `explain select` is available in Aerospike Tools package 3.6.1 and above.

The following command displays details for a single record:

```ini
explain select * from <ns>.<set> where PK=<primary key value>
```

Example:

```sql
aql> explain select * from test.uqr where PK=1
[
{
"SET": "uqr",
"DIGEST": "42 7A 64 AE 7B 80 EE 02 3D B8 E9 00 8A F3 FF B0 7F DA EB 58",
"NAMESPACE": "test",
"PARTITION": 2626,
"STATUS": 2,
"UDF": "FALSE",
"KEY_TYPE": "STRING",
"TIMEOUT": 1000,
"NODE": "1771FE6C15290C00",
"POLICY": "AS_POLICY_REPLICA_MASTER",
"KEY_POLICY": "AS_POLICY_KEY_SEND"
}
]
```

Where:
 - “SET” is the set name for the record.
 - “DIGEST” is the digest for the record.
 - “NAMESPACE” is the namespace for the record.
 - “PARTITION” is the partition where the record lives. The partition value is between 1 and 4095.
 - “STATUS” is the status of the operation. It will be the code as returned by the client for the record. E.g: 0 is `AEROSPIKE_OK` and 2 is `AEROSPIKE_ERR_RECORD_NOT_FOUND`. See here for list of generic error codes in C - [Error Codes](https://github.com/aerospike/aerospike-client-c/blob/master/src/include/aerospike/as_status.h).
 - “UDF” tells if the AQL command was run as part of a UDF.
 - "KEY_TYPE" is the type of the key (string/numeric etc) that was sent to the server.
 - “TIMEOUT” is the timeout for the AQL command.
 - “NODE” is the ID of the node where the master record currently lives. The node ID might change during migrations.
 - “POLICY” is the replica read policy that the client can set when reading the record. Two possible values are:<br>
 1) `AS_POLICY_REPLICA_MASTER` (which is by default) to set the reads to go only to it's master copy and, <br>
 2) `AS_POLICY_REPLICA_ANY` to read in a roundrobin operation on all it's replica copies. Set the policy to `AS_POLICY_REPLICA_ANY` by executing `set replica_any true` in AQL.
 - “KEY POLICY” indicates whether the client had sent the key and digest to the server or just the digest. Two possible values are:<br>
 1) `AS_POLICY_KEY_SEND` if the key was sent along with the digest, and<br>
 2) `AS_POLICY_KEY_DIGEST` if only the digest was sent.
