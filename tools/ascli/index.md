---
title: ascli
description: Learn how ascli provides a set of commands (get, put, remove, etc) that can be executed against an Aerospike database.

breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
---

{{#note}}
Deprecated from Aerospike Tools Versions >= 3.9.0. Use aql instead.
{{/note}}

Ascli provides a set of commands (get, put, remove, etc) that can be executed against an Aerospike database.

### Usage

The basic usage of ascli is:

```bash
ascli OPTIONS COMMAND
```

### Options

The following are options for ascli:

- `--path=<path>`

  Specify the search path for ascli commands (C executables)

- `--list`

  List available commands

- `--help`

  Show help message


### Commands

The following are commands for ascli. Each command has its own set of options and arguments. See the command's documentation for details.

|Command|Description|
|---|---|
|`exists`|  Check if a record exists.|
|`get`|  Get a record.|
|`put`|  Put a record.|
| `remove`|  Remove a record.|
| `query-list`| List the query jobs currently running.|
| `query-kill`| Kill a query job.|
| `scan-list`| List the scan jobs currently running.|
| `scan-kill`| Kill a scan job.|
| `udf-get`| Download a UDF module.|
| `udf-put`|  Upload a UDF module.|
| `udf-list`|  List UDF modules.
|`udf-remove`|  Remove a UDF module.|
| `udf-record-apply`|  Apply a UDF to a record.|
