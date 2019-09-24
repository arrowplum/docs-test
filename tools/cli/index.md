---
title: Command-Line Utility (cli)
description: Learn how cli, the command line utility, can be used to perform get, set and delete operations.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
---

{{#note}}
Deprecated from Aerospike Tools Versions >= 3.9.0.
{{/note}}

The Aerospike Command Line Tool (cli) allows simple command line execution of basic get, set, and delete commands to validate basic database operations.
You can get the usage information of the command line tool as follows:

### Usage

Execute a command against an Aerospike cluster.

```bash
cli OPTIONS
```

### Options

- `-t, --target <host:port>`

  a cluster node to query (host:port)
  
  default: 127.0.0.1:3000

- `-h, --host <host>`

  host of the cluster node to query
  
  default: 127.0.0.1

- `-p, --port <port>`

  port of the cluster node to query
  
  default: 3000

- `-U, --user  <user name>`

  User name to access record, required when using the enterprise security feature.
  
- `-P, --password [password]`

  Password to access record, required when using the enterprise security feature.

- `-o, --operand <operand>`

  Database operation, note that set and put are equivalent: get, set, put, delete
  
  default: get

- `-n  --namespace <ns>`

  the namespace to operate on
  
  default: test

- `-s  --set <set>`

  the set the key belongs in

  default: ""

- `-k, --key`

  key - must be set, no default

- `--d64, --digest-base64`

  Interpret key as a base64 digest
  
- `--d16, --digest-base16, --digest-hex`

  Interpret key as a base16 (hex) digest

- `-i, --integer-key, --key-integer`

  True if key is a integer

- `-b, --bin`

  name of the bin

  default: ""

- `-e, --recordttl <ttl>`

  record TTL
  
  default: None

- `-v, --value`

  value

- `--verbose`

  make the request verbose

### Examples

Set an object with a key string of "server01" and a value of "This is my hostname" in the namespace "users"

```bash
cli -o set -n users -k server01 -v "This is my hostname"
```

Get a records with a key of "server01" from the namespace "users"

```bash
cli -o get -n users -k server01
```

Note that this tool is meant to be used for basic validation only. Aerospike is not intended to be used through the command line tool. cli creates a new connection pool for every transaction which is very inefficient.
