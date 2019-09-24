---
title: aql – Query and Scan Management
description: Learn how the aql command can be used to  manage queries or scans by listing or killing running queries or scans.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: aql
    url: /docs/v3/aql.html
---

### List Running Jobs


Tracking of secondary index queries automatically starts if the query takes more than the configured [`query-untracked-time-ms`](/docs/reference/configuration/#query-untracked-time-ms). Note that with queries, the server cannot estimate the size of the result and therefore, the completion percentage is not tracked.


The following is the command to list running queries (See [Managing Queries](/docs/operations/manage/queries)): 
```sql
SHOW QUERIES
```

The following is the command to list running scans (See [Managing Scans](/docs/operations/manage/scans)):
```sql
SHOW SCANS
```

This will display a list of query or scan jobs, with their identifiers and statistics. You can use the identifiers to kill a specific job.

### Kill Running Jobs
The following is the command to kill a running query:
```sql
KILL_QUERY <id>
```

The following is the command to kill a running scan:
```sql
KILL_SCAN <id>
```

The `<id>` parameter is the query identifier as in SHOW QUERIES or SHOW SCANS.
