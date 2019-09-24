---
title: ascli – Key-Value Store Operations
description: Learn the ascli commands for reading and writing records, including put, get, exists and remove.
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

The following are ascli commands for reading and writing records.

For details of the command line options, look at the overview of the ascli command.

### put
To put a record in the database, use the put command:

```bash
ascli put <ns> <set> <key> <record>
```

The `<ns>` is the namespace for the record being written.

The `<set>` is the set for the record being written.

The `<key>` is the key for the record being written. It can be either an integer or string.

The `<record>` is a JSON object representing the record to be written. The JSON object's keys are the bin names and the value are bin values.

The values will be either an Integer, String, List, or Map. The List is encoded as a JSON Array. The Map is encoded as a JSON Object.

Success is indicated by a "0" exit code. Any other exit code indicates a failure.

Example:

```bash
$ ascli put test test test '{"a": "A", "b": 1, "c": [1,2,3], "d": {"x": 4}}'
$ if [ $? == 0 ]; then echo "success"; else echo "failure"; fi;
success
```

### get

To get a record from the database, use the get comamand:

```bash
ascli get <ns> <set> <key>
```

The `<ns>` is the namespace for the record being read.

The `<set>` is the set for the record being read.

The `<key>` is the key for the record being read. It can be either an integer or string.

The result will be a JSON object representing the record read, where the keys are bin names and the value are bin values. 

The values will be either an Integer, String, List, or Map. The List is encoded as a JSON Array. The Map is encoded as a JSON Object.

Example:

```bash
$ ascli get test test test
{"a": "A", "b": 1, "c": [1,2,3], "d": {"x": 4}}
```


### exists

To test the existence of a record in the database, use the exists command:

```bash
ascli exists <ns> <set> <key>
```

The `<ns>` is the namespace for the record being tested.

The `<set>` is the set for the record being tested.

The `<key>` is the key for the record being tested. It can be either an integer or string.

Existence of the record is indicated by a "0" exit code. Any other exit code indicates that the record does not exist.

Example:

```bash
$ ascli exists test test test
Key: test exists.
$ if [ $? == 0 ]; then echo “exists”; else echo “doesn't exist”; fi;
exists
```

### remove

To remove a record from the database, use the remove command:

```bash
ascli remove <ns> <set> <key>
```

The `<ns>` is the namespace for the record being removed.

The `<set>` is the set for the record being removed.

The `<key>` is the key for the record being removed. It can be either an integer or string.

Success is indicated by a "0" exit code. Any other exit code indicates either a failure or that the record did not exist.

Example:

```bash
$ ascli remove test test test
$ if [ $? == 0 ]; then echo “success”; else echo “failure”; fi;
success
$ ascli exists test test test
$ if [ $? == 0 ]; then echo “exists”; else echo “doesn't exist”; fi;
doesn't exist
```
