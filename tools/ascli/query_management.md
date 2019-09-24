---
title: ascli – Query Management
description: Learn about the list and kill ascli commands for managing query jobs.

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

The following are ascli commands for managing query jobs.

{{#note}}{{#markdown}}
	

For query job management query tracking needs to be enabled. It is disabled by default to avoid overhead for the high through low latency queries.

To enable query tracking:
asinfo -v "set-config:context=service;query-job-tracking=true"

To disable query tracking:
asinfo -v "set-config:context=service;query-job-tracking=false"

{{/markdown}} {{/note}}

### query-list

This command displays a list of any queries running on the cluster.  If you specify a node/port, then only the queries running on the specified node are listed.  The syntax of this command is:

```bash
ascli query-list [-v]
```

The `-v` option will print a more detailed listing.

Example:

```bash
$ ascli query-list
TRANSACTION ID          STATUS
472341955492950239      RUNNING
```

If using verbose mode, the results look like this:

```bash
$ ascli query-list -v
NODE               TRANSACTION ID          STATUS     PROGRESS      EXEC TIME(ms)
BB92A20F7290C00    13912474952720922700    RUNNING    [n=191079]    24672587
```

### query-kill

This command terminates a query.  The syntax is:

```bash
ascli query-kill <id>
```

The `<id>` is obtained from the query-list command. 

Example:

```bash
$ ascli query-kill 14542284165954543608
TRANSACTION ID          STATUS
14542284165954543608    Ok
```
For more information see [Managing Queries](/docs/operations/manage/queries)	

