---
title: Examples
description: The Aerospike C client library bundle contains examples that illustrate building applications. 
---

The Aerospike C client library source code [repository](https://github.com/aerospike/aerospike-client-c)
contains examples that illustrate building applications. These examples are in the _examples_ directory.

Examples include:
- Basic examples
 - `put` and `get` a record with multiple bins.
 - Set the TTL (time-to-live) for a record.
 - Apply a UDF on a record.
- Batch examples
 - Use a aingle request for multiple records.
- Query examples
 - Secondary index query and aggregation.
- Scan examples
 - Scan for a whole namespace or only a set.
 - Scan and apply a UDF on the result set.

### Linux/MacOS

To build all examples on Linux/MacOS, get the source code from [git repo](https://github.com/aerospike/aerospike-client-c) and run `make` from the examples directory:

```bash
$ git clone https://github.com/aerospike/aerospike-client-c.git
$ cd aerospike-client-c
$ git submodule update --init
$ make
$ cd examples
$ make
```

On a successful build, run the examples:

```bash
$ make run
```

To build and run specific examples, provide the path to the example:

```bash
$ make -C basic_examples/put 
$ make -C basic_examples/put run
```

{{#note}}
You can run `make` from any example directory.
{{/note}}

To clean up the examples, run the `clean` target:

```bash
$ make clean
```

Refer to the README file in the _examples_ directory for more information. Connection examples are in the [Aerospike C client Library](/docs/client/c/usage/connect).

### Windows

The examples project is included the aerospike solution in the source code
[repository](https://github.com/aerospike/aerospike-client-c).

- Double-click `vs/aerospike.sln`
- Click `Build -> Build Solution`
- Click `examples`
- Right-click on an example and choose `Set as StartUp Project`
- Right-click on same example and choose `Properties`
- Click `Debugging`
- Enter `-h {your server hostname}` in `Command Arguments` field.
- Click `OK`
- Press `{control}{F5}`

The example should run in a console window.
