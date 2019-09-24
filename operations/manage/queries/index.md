---
title: Managing Queries
description: Learn how you tune various parameters for queries across an Aerospike cluster. 
---

{{#todo markdown=true}}
- Rewrite documentation to use AQL instead of asinfo
{{/todo}}

### Tuning Queries

Aerospike allows you to tune various parameters for queries across a cluster, including:

- The number of threads allocated to queries.
- The ability to efficiently process small data sets.

The following table specifies the subsystem level configurations you can modify with the asinfo tool.

Name	| Value	| Default |	Comments
--- | --- | --- | ---
query-in-transaction-thread | 1/0 | 0 |Set this to '1' when you expect the query to run for a short duration or if you are using an in-memory namespace. Set this to '0' if you expect the query to run for a longer duration or you are using a disk as storage.
query-priority-sleep-us | 0 ... UINT64_MAX | 1 | Controls the time that the server pauses between queries. A lower value increases the number of queries that run. More queries on the namespace might affect read/write speed.  Value is in microseconds.
query-priority | 0 ... UINT64_MAX | 10 | You can change the priority for queries by increasing or decreasing the query-priority.
query-threads | 1 ... 32 | 6 | Number of dedicated query threads in the system.
query-worker-threads	| 0 ... 480 | 15 | Number of dedicated query I/O threads.
query-batch-size | 0 ... UINT_MAX | 100 | Batch size indicates the amount of disk I/O query performs per I/O request. Keep this value low to avoid I/O bottlenecks.
query-req-max-inflight | 0 ... UINT_MAX | 100 | Limit on the number of query I/O threads used per query at a time. A higher value might affect shorter queries. 
query-long-q-max-size | 0 ... UINT64_MAX | 500 | Number of queries in the long running query queue. A long running query is one that returns more records than the query-threshold. If you change this value while a query is running, the change does not affect the running query.
query-short-q-max-size | 0 ... UINT64_MAX | 500 | Number of queries in the short running query queue. A short running query is one that returns fewer records than the query-threshold. If you change this value while a query is running, the change does not affect the running query.
query-threshold | 0 ... UINT64_MAX | 10 | Dividing line between short running and long running queries. A query that returns fewer records than the query threshold is a short running query. All others are long running queries.

### Updating Query Settings

The above parameters can be dynamically set in the cluster using [Aerospike Info (asinfo)](/docs/tools/asinfo), using the following command:

```
asinfo -v "set-config:context=service;<name>=<value>"
```

Where <name> is the configuration parameter name, and <value> is the parameter value.

#### Query Job Management

Tracking of secondary index queries automatically starts if the query takes more than the configured [`query-untracked-time-ms`](/docs/reference/configuration/#query-untracked-time-ms). Note that with queries, the server cannot estimate the size of the result and therefore, the completion percentage is not tracked.

Below is the description of the configurations variables. 

**Query Job Priority:**	

The value for priority must be greater than or equal to '1'.

```
asinfo -v 'jobs:module=query;cmd=set-priority;trid=<jobid>;value=100'
```

**Query Job Kill:**	

Use the following syntax to kill a running query:

```
asinfo -v 'jobs:module=query;cmd=kill-job;trid=<jobid>'
```

### List Queries
 
 Use the following syntax to list queries:
 
```sql
aql> show queries
```

Fields:

- **status**

  - **IN_PROGRESS** -  Indicates the jobs in progress.
  - **DONE** - Indicates the job is complete.
  - **ABORT** - Indicates the job was either aborted or killed by a user.

- **mem_usage**

  Amount of memory in bytes used at any time. The query writes results back to the client in batched manner and controls the amount of memory used. Higher value of this should be watched.

- **run_time**

  Duration of the running query in microseconds.

- **recs_read**

  Number of records read by this query. Multiple simultaneous reads and writes on a namespace can affect performance.

- **net_io_bytes**

  Number of bytes the scan job returns to the client. Query related network bytes transfer.

- **priority**

  Priority of this query. The priority may be increased or decreased.
  
- **indexname**

  Name of the index for the query.  

### Important statistics to monitor

```sql
aql> stat system
```

Fields:

- **query_success**

  Number of successful queries on the cluster. Includes the number of lookups and aggregations.

- **query_avg_selectivity**

  Average selectivity of all the aggregations which ran in the cluster. Includes the number of lookups and aggregations.

- **query_agg_success**

  Number of successful aggregations on the cluster.   

- **query_agg_avg_selectivity**

  Average number of records selected by each query over the lifetime of this server.

- **query_lookup_avg_success**

  Number of successful lookups that ran on the cluster.

- **query_lookup_avg_selectivity**

  Average number of records returned by the lookups over the lifetime of this server.
 
### Query Latency

A overall query histogram is written to the log file every 10 seconds.

```
Sep 28 2013 14:58:56 GMT: INFO (info): (src/main/hist.c:55) histogram dump: query (267911 total)
Sep 28 2013 14:58:56 GMT: INFO (info): (src/main/hist.c:70)  (00: 0000238581)  (01: 0000024013)  (02: 0000003574)  (03: 0000000963) 
Sep 28 2013 14:58:56 GMT: INFO (info): (src/main/hist.c:70)  (04: 0000000572)  (05: 0000000174)  (06: 0000000033)  (07: 0000000001)
```

### Query micro benchmarks

Use the following syntax to enable writing micro benchmarks to the logs:

```
asinfo -v "set-config:context=service;query-microbenchmark=true"
```

Use the following syntax to stop writing micro benchmarks to the logs:

```
asinfo -v "set-config:context=service;query-microbenchmark=false"
```
Use the following syntax to enable secondary index specific micro benchmarks:

```
asinfo -h [host ip] -v "sindex-histogram:ns=NAMESPACE;indexname=INDEX;enable=true"
```

Use the following syntax to stop writing secondary index specific benchmarks to the logs:

```
asinfo -h [host ip] -v "sindex-histogram:ns=NAMESPACE;indexname=INDEX;enable=false"
```

### Recommendations

Enable tracking only if you run long running queries or for debugging.
By default, query_in_transaction_thread and query_req_in_query_thread are set to '0'. Set them to '1' when the database is in memory or when the query returns fewer records.

### Related Links

[aql – Query and Scan Management](/docs/tools/aql/query_scan_management.html) 
