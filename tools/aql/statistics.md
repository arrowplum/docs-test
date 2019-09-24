---
title: aql – Statistics
description: Learn how aql can be used to list statistics on the database, read the statistics for a secondary index or list statistics for queries.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: aql
    url: /docs/v3/aql.html
---

### System Statistics

The following command will list statistics on the database:

```sql
STAT SYSTEM
```

Example:

```sql
aql> stat system
+---------------------------------+--------------------+
| name                            | value              |
+---------------------------------+--------------------+
| "cluster_size"                  | 1                  |
| "cluster_key"                   | "AA47E9295DD25BC2" |
| "cluster_integrity"             | "true"             |
| "objects"                       | 12                 |
...
179 rows in set (0.000 secs)
```

### Secondary Index Statistics

The following is the command to read the statistics for a secondary index:

```sql
STAT INDEX <ns> <index>
```

This will get the statistics of the `<index>` index in `<ns>` namespace.

Where

- `<ns>` is the namespace containing the index to be checked.
- `<index>` is the name of the index to be checked.

The following is an example of checking the stats on the index named "numindex" in "test" namespace.

```sql
aql> stat index test numindex
+--------------------------+-------+
| name                     | value |
+--------------------------+-------+
| "keys"                   | 6     |
| "objects"                | 11    |
| "data_memory_used"       | 1320  |
| "load_pct"               | 100   |
| "loadtime"               | 6     |
| "stat_write_reqs"        | 11    |
| "stat_write_success"     | 11    |
| "stat_write_errs"        | 0     |
| "stat_delete_reqs"       | 0     |
| "stat_delete_success"    | 0     |
| "stat_delete_errs"       | 0     |
| "stat_defrag_recs"       | 0     |
| "stat_defrag_time"       | 0     |
| "n_query"                | 6     |
| "avg_selectivity"        | 2     |
| "avg_record_size"        | 42    |
| "n_aggregation"          | 5     |
| "agg_avg_selectivity"    | 2     |
| "agg_avg_record_size"    | 8     |
| "n_lookups"              | 1     |
| "lookup_avg_selectivity" | 6     |
| "lookup_avg_record_size" | 106   |
+--------------------------+-------+
22 rows in set (0.000 secs)
```

### Namespace Statistics

The following is the command to list statistics for namespaces:

```sql
STAT NAMESPACE <ns>
```

This will get the statistics of the `<ns>` namespace.

Where

- `<ns>` is the namespace to be checked.

The following is an example of checking the stats on the "test" namespace.

```sql
aql> stat namespace test
+---------------------------------------+----------------+
| name                                  | value          |
+---------------------------------------+----------------+
| "ns_cluster_size"                     | "1"            |
| "effective_replication_factor"        | "1"            |
| "objects"                             | "985178"       |
| "tombstones"                          | "0"            |
| "master_objects"                      | "985178"       |
| "master_tombstones"                   | "0"            |
| "prole_objects"                       | "0"            |
| "prole_tombstones"                    | "0"            |
| "non_replica_objects"                 | "0"            |
| "non_replica_tombstones"              | "0"            |
| "dead_partitions"                     | "0"            |
| "unavailable_partitions"              | "0"            |
| "clock_skew_stop_writes"              | "false"        |
| "stop_writes"                         | "false"        |
| "hwm_breached"                        | "false"        |
| "current_time"                        | "289037042"    |
| "non_expirable_objects"               | "0"            |
| "expired_objects"                     | "2425626"      |
| "evicted_objects"                     | "22931"        |
| "evict_ttl"                           | "85140"        |
| "evict_void_time"                     | "289018038"    |
| "smd_evict_void_time"                 | "281648962"    |
| "nsup_cycle_duration"                 | "0"            |
| "truncate_lut"                        | "265414455159" |
| "truncated_records"                   | "16722909"     |
| "memory_used_bytes"                   | "63069824"     |
...
+--------------------------------------------+------------+
[127.0.0.1:3000] 217 rows in set (0.004 secs)
```

