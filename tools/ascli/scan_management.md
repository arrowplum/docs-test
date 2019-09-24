---
title: ascli – Scan Management
description: Learn about the list and kill ascli commands for managing scan jobs.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: Aerospike CLI (ascli)
    url: /docs/v3/ascli.html
---

{{#note}}
Deprecated from Aerospike Tools Versions >= 3.9.0. Use aql instead.
{{/note}}

The following are ascli commands for managing scan jobs.


### scan-list

This command displays a list of any scan UDFs running on the cluster.  If you specify a node/port, then only the scan UDFs running on the specified node are listed.  The syntax of this command is:

```bash
ascli scan-list
```

Example:

```bash
$ ascli scan-list
SCAN ID                 STATUS     PROGRESS    PRIORITY    EXEC TIME(ms)  NS/SET
2137924950893220379     IN PROG    0%[n=63]    medium      1526           test/null
```

### scan-kill

This command terminates a scan UDF.  The syntax is:

```bash
ascli scan-kill <id>
```

The `<id>` is obtained from the scan-list command. 

Example:

```bash
$ ascli scan-kill 2137924950893220379
SCAN ID            STATUS
2137924950893220379     Ok
```
