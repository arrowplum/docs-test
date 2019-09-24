---
title: Managing Indexes
description: Leverage Aerospike tools for creating, managing, repairing, or dropping indexes.
---

Aerospike provides tools for managing secondary indexes. These include :

- [AQL](/docs/tools/aql), which uses a SQL-like interface.
- [Aerospike Info (asinfo)](/docs/tools/asinfo), interacts with the server using the native info protocol.


### Create and Manage Index

AQL provides the ability to create indexes using a SQL-like syntax:

```sql
CREATE INDEX ind1 ON test.demo (bin1) NUMERIC
```

For details on this command, see [aql – Index Management](/docs/tools/aql/index_management.html).

### Listing Indexes

Use AQL to list indexes in the database:

```sql
aql> show indexes
+--------+--------+-----------+----------+--------+-----------+------------+-------+
| ns     | set    | indexname | num_bins | bins   | type      | sync_state | state |
+--------+--------+-----------+----------+--------+-----------+------------+-------+
| "test" | "demo" | "ind2"    | 1        | "bin2" | "NUMERIC" | "synced"   | "RW"  |
| "test" | "NULL" | "ind3"    | 1        | "bin2" | "STRING"  | "synced"   | "RW"  |
| "test" | "demo" | "ind4"    | 1        | "bin3" | "STRING"  | "synced"   | "RW"  |
| "abcd" | "demo" | "ind1"    | 1        | "bin1" | "NUMERIC" | "synced"   | "RW"  |
+--------+--------+-----------+----------+--------+-----------+------------+-------+
```

For more information on indexes, see [aql – Index Management](/docs/tools/aql/index_management.html). 

Use AQL to list indexes in a namespace:

```sql
aql> show indexes abcd
+--------+--------+-----------+----------+--------+-----------+------------+-------+
| ns     | set    | indexname | num_bins | bins   | type      | sync_state | state |
+--------+--------+-----------+----------+--------+-----------+------------+-------+
| "abcd" | "demo" | "ind1"    | 1        | "bin1" | "NUMERIC" | "synced"   | "RW"  |
+--------+--------+-----------+----------+--------+-----------+------------+-------+
```

### Dropping Indexes

Use AQL to drop indexes from the cluster:

```
DROP INDEX test ind1
```

For more information on the DROP command, see [aql – Index Management](/docs/tools/aql/index_management.html).

    
{{#info}} Index DDL changes are synchronous operations. When you run an index command, the nodes in the cluster consult each other and then run the command on each node. When the command is complete, verify the state of the index before using it.{{/info}}
    
{{#info}}You should consider whether or not you need a secondary index before creating one.  You should avoid creating several indexes at one time, or creating indexes while nodes are migrating.{{/info}}


### Repairing Indexes

{{#note}}
Deprecated in versions 3.14 and above.
{{/note}}

If you find an index with sync_state=need_sync and state=RW, you should use AQL to repair it.

Use [Aerospike Info (asinfo)](/docs/tools/asinfo) to repair the index:

```
$ asinfo -v "sindex-repair:ns=test;indexname=ind_name;set=set_name;"
```

### Static Configuration for Index Management. These configurations are deprecated in version 3.14 and above.

If there are index-specific tuning configurations that you would like to persist, you may add them in a "si" stanza in the specific namespace. 

You may add the following parameters to aerospike.conf:


Name | Description | Minimum | Maximum | Default | Deprecated |
--- | --- | --- | --- | ---
si-gc-period    | Interval between two iterations of index garbage collection. Value is in milliseconds. | 0   | -  |1000 | >3.14
si-gc-max-units |   Maximum number of elements reviewed in each garbage collection cycle.   | 0 | - | 1000 | >3.14
si-data-max-memory | Maximum amount of memory used by a specific index. Value is in bytes.  | 0 | ULONG_MAX | ULONG_MAX | >3.11
si-tracing |    Tracing level for this index. | 0 | 4 | 0 | >3.11
si-histogram |  Enable and disable si-histogram mapping of this index for reads and writes in the server log.    | - | - |   False | >3.14

You can edit the settings in the namespace section of the configuration file. 

The following example shows the static configuration for the secondary index ind1:
 
```
namespace test {
    ...
    si ind1 {
       si-gc-period 100
       si-gc-max-units 10000
       si-tracing 0
       si-histogram false
    }
}
``` 
Note: You must create the secondary index before you add the stanza to aerospike.conf. 

### Index Memory Management

Aerospike Secondary indexes exist entirely in memory. If you use secondary indexes, verify that the nodes in the cluster have enough memory for the indexes. For more information on memory allocation for secondary indexes, see [capacity planning](/docs/operations/plan/capacity).

If the namespace size is not configured to take into account the memory requirements of secondary indexes, and the memory high water mark is breached, the cluster may evict data or stop writes. You can limit the amount of memory used by secondary indexes to prevent the namespace from running out of memory. If it runs out of memory, the database stops updates to the secondary index and changes it's sync_state to 'need_sync'. In this case, you can repair the index and add more memory for the index. If the secondary index's sync_state is changed to 'need_sync', it may return stale records when queried.

- `sindex-data-max-memory`

  Deprecated in version 3.11 and above. Global configuration that limits the amount of memory for secondary indexes on a node. The value is in bytes.

  Default: ULONG_MAX

  ```bash
  asinfo -v "set-config:context=service;sindex-data-max-memory=<value>"
  ```

- `sindex-data-max-memory`

  Deprecated in version 3.11 and above. Namespace configuration that limits the amount of memory for secondary indexes for a namespace. The value is in bytes.

  Default: ULONG_MAX

  ```
  asinfo -v "set-config:context=namespace;id=<namespace>;sindex-data-max-memory=<value>"
  ```

- `data-max-memory`

  Deprecated in version 3.11 and above. Index configuration that limits the amount of memory for a single secondary index. The value is in bytes. 

  ```
  asinfo -v "set-config:context=namespace;id=<namespace>;indexname=<iname>;data-max-memory=<value>"
  ```

- `sindex-populator-scan-priority`

  Deprecated in version 3.6 and above. When secondary indexes are created, the nodes scan data to load secondary indexes. `sindex-populator-scan-priority` defines the priority of the background scan process.

  Default: 3

  ```
  asinfo -v 'set-config:context=service;sindex-populator-scan-priority=5'
  ```

For details of other configuration options, see the [Configuration Reference Guide](/docs/reference/configuration).

### Index Monitoring
 

#### Important statistics to monitor:

```sql
stat index <namespace> <indexname>
```

NAME | DESCRIPTION | Comments
--- | --- | ---
keys    | Number of primary keys indexed to this index. | Statistics to monitor the quantity of unique keys in a secondary index.
entries | Number of objects in the secondary index tree.| For unique indexes, this value should equal the number of keys.
ibtr_memory_used |  Total memory used by a secondary index.  | Use this to check system memory memory usage.
nbtr_memory_used |  Total memory used by a secondary index.  | Use this to check system memory memory usage.
stat_gc_recs |  Number of records garbage collected from the secondary index.   |
stat_gc_time |  Total time taken to cleanup garbage from the secondary index.   |
query_reqs |    Cumulative number of query requests on a given secondary index. (query_lookups + query_agg)  |
query_avg_rec_count | Average of number of records selected over the number of queries on a given secondary index. |
 
 
#### Displaying Performance Histograms for Secondary Indexes

  To enable secondary index performance histograms:
  ```bash
asinfo -v "sindex-histogram:ns=<namespace>;[set=demo];indexname=<index name>;enable=true"
  ```
  To disable secondary index performance histograms:
  ```bash
  asinfo -v "sindex-histogram:ns=<namespace>;[set=demo];indexname=<index name>;enable=false"
  ```

'enable=true' writes the secondary index performance histogram to the log.

'enable=false' stops writing the secondary index performance histogram to the log.

#### Displaying Performance Histograms for Secondary Indexes Garbage Collection

  Deprecated in version 3.14 and above. 

  To enable secondary index garbage collection performance histograms:
  ```bash
asinfo -v "set-config:context=service;sindex-gc-enable-histogram=true"
  ```
  To disable secondary index garbage collection performance histograms:
  ```bash
  asinfo -v "set-config:context=service;sindex-gc-enable-histogram=false"
  ```

'enable=true' writes the secondary index garbage collection histogram to the log.

'enable=false' stops writing the secondary index garbage collection histogram to the log.

#### Index Limits

The index name must be 255 or less characters in length, and not include colon (:) or semicolon (;).

The maximum size of a string that can be indexed is 2KB.

The total number of indices per namespace is 256.


