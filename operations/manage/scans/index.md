---
title: Managing Scans
description: Aerospike provides a set of configuration parameters and an API to manage scans. Find out how you can manage scans properly. 
---

{{#todo markdown=true}}
- Rewrite documentation to use AQL instead of asinfo
- Rewrite similar structure to Queries (assuming queries has been rewritten as well)
{{/todo}}

Aerospike scans are long-running, resource intensive jobs. These jobs walk through an entire namespace and:

- Read records, taking up I/O resources.
- May run UDFs (scan UDF) on records, taking up CPU resources.
- May return large amounts of data to the client (e.g. the backup tool), taking up network bandwidth.

Given that scans can effect all three major system resources, it is imperative that the scan subsystem is configured and scan jobs managed properly.

Aerospike provides a set of configuration parameters and an API to manage scans. For more information regarding their default values, please see [Configuration Reference](/docs/reference/configuration).

# Server version 3.6.0 and above

Scans can be initiated in parallel and multiple scans are interleaved by default. Additionally, scans with higher priority will be processed on a separate priority queue to be able to finish faster than lower priority scans in addition to being actively running in parallel.

### Manage

Maximum number of active scans that can be initialized concurrently:

```
asinfo -v 'set-config:context=service;scan-max-active=10'
```

Maximum number of finished scans that can be kept around for monitoring:

```
asinfo -v 'set-config:context=service;scan-max-done=10'
```

Configure number of scan threads to throttle the scan performance:

```
asinfo -v 'set-config:context=service;scan-threads=5'
```


# Set priority and kill scan jobs

**Scan Job Priority:**

Scan jobs can be run at three different priorities: Low (value=1), Medium (value=2), and High (value=3).

```
asinfo -v 'jobs:module=scan;cmd=set-priority;trid=<jobid>;value=3'
```

Default priority is `2` (Medium). The scan job priority may also be specified when initiating the scan job.

---

**Scan Job Kill:**  

The following commands will cause running scan jobs to be aborted. To see the list of jobs and the job ID, use `asinfo -v 'jobs:'`.

```
asinfo -v 'jobs:module=scan;cmd=kill-job;trid=<jobid>'
asinfo -v 'scan-abort:id=<jobid>'
```

To kill all on-going scans, use the following syntax (available for Aerospike release 3.5.12 onwards).

```
asinfo -v 'scan-abort-all:'
OK - number of scans killed: 1
```

---

**SIndex Scan Job Priority:**

Sindex populators are background scan jobs which run with default priority of medium. Update `sindex-populator-scan-priority` to accelerate an index loading process for versions 3.5.15 and older.

```
asinfo -v 'set-config:context=service;sindex-populator-scan-priority=5'
```

For version 3.6.0 and above, to accelerate the sindex loading process, configure Secondary index builder threads.

```
asinfo -v 'set-config:context=service;sindex-builder-threads=5'
```

For details about managing secondary indexes, please see [Manage Indexes](/docs/operations/manage/indexes).

---

# List Scans

Aerospike scans are long running slow operations which generally run in the background. A scan can be considered as a type of job. They can be monitored either using Aerospike Management Console or using the command line.

```sql
aql> show scans
```

Fields:

- **trid**

  ID number of the scan job.

- **job-type**

  - **basic** - Simple data scan job.
  - **aggregation** - Aggregation scan jobs, which aggregate query results and return them to the client.
  - **background-udf** - Background jobs which run in the detached mode and apply a UDF on scanned records. They do not return any data back to client.
  - **sindex-build** - Data scan job which is triggered by index creation to load data into a secondary index. It is a background job.
  - **sindex-build-all** - Data scan job initiated during server startup to load data into all secondary indexes. It is a background job.

- **ns**

  Namespace associated with the scan job.

- **set**

  Set name (if any) associated with the scan job.

- **priority**

  Indicates the priority of the job. This priority may be toggled up and down as required. High priority would mean scan would consume more resources. Scan priority should be set according to system load.

- **status**

  - **active(ok)** – In progress
  - **done(ok)** – Completed with no error
  - **done(user-aborted)** – Scan aborted by user
  - **done(abandoned-unknown)** – Scan aborted with unknown failure
  - **done(abandoned-cluster-key)** – Scan aborted due to cluster key change
  - **done(abandoned-response-error)** – Scan aborted due to response error
  - **done(abandoned-response-timeout)** – Scan aborted due to timeout

- **job-progress**

  Gives a rough estimate of job completion percentage.

- **run-time**

  Run duration of the job in ms. A larger value means that the job is taking longer to finish.

- **time-since-done**

  Elapsed time since the job finished, in ms.  This value is zero if the job is still running.

- **recs-read**

  Number of records read by the scan job. 

- **net-io-bytes**

  Amount of bytes being sent back over wire to client by this scan job. Basically, the scan related network bytes transfer and related I/O being issued. This could have a possible effect on normal reads and writes. Consider reducing scan job priority to reduce the I/O load.

- **socket-timeout** (server version 4.5.2 or later)

  The socket timeout for the scan job, in ms.
  This value can be specified by the client for basic scans and aggregations.  For background UDF scans, it is the server default of 10000 (10 seconds).
  For secondary index jobs, it is zero.

- **from** (server version 4.5.2 or later)

  For all scans other than secondary index jobs, the client address associated with the scan.

- **sindex-name**

  Name of secondary index for sindex-build jobs.

- **udf-filename, udf-function**

  For background UDF scans, the UDF filename and the function name of the UDF being run.

- **udf-active**

  For background UDF scans, the number of active UDF transactions.

- **udf-success, udf-failed**

  For background UDF scans, the number of successful and failed UDF transactions.

# Recommendations
- Scans are designed for running a few long-running jobs, not for many concurrent scans. The scan subsystem is also used by:
  - The secondary index populator.
  - The backup utility.
  Caution must be exercised to make sure the system does not get loaded with lots of parallel scans.

- Scans should be avoided while performing cluster administrative activity like node addition, removal etc. Scans do not return precise results when migrations are going on.

- Scans should generally be run at low load time to avoid affecting normal runtime read/write throughput. If required in such circumstances, scans should be run with care.

# Server version 3.5.15 and before

When a scan is initiated it is queued in the global scan job queue. This queue is serviced by a scan thread which creates multiple small scan jobs and assigns them to worker threads. These worker threads do the real job of performing I/O, network writes / queuing up UDF transactions / updates of secondary indexes, based on the type of scan job. Once the job is done the entry is removed from global queue.

### Manage

**Global Scan Priority**

This, along with global scan sleep, controls the resources used by the system.

```
asinfo -v 'set-config:context=service;scan-priority=10'
```

This parameter defines the scans per yield. The system will run scans at the lowest priority when `scan-priority` is set to 1.
Value should be left to default unless there is a specific need to run the scan at much higher or much lower priority.


**Global Scan Sleep**

This parameter determines the sleep time per yield in milliseconds.

```
asinfo -v 'set-config:context=service;scan-sleep=10'
```

They system will sleep more between each yield when `scan-sleep` is increased. Value should be left to default unless there is a specific need to run the scan at much higher or much lower priority.

---
