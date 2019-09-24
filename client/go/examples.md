---
title: Examples
description: Source code examples demonstrate using the Aerospike Go client and the Aerospike database.
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
---

Examples demonstrating Aerospike Go client usage are in the _examples_ directory in **<a id="github" href="{{book.vars.github-url}}">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a>**

Examples include:

Example | Description
--- | ---
Server Info | Use Aerospike info protocol to query a server node for statistics.
Put/Get | Write and read a record.
Replace | Write bins with replace option.
Add | Perform a server integer add.
Append  | Perform a server string append.
Prepend | Perform a server string prepend.
Batch | Perform multiple exists, get, or get header commands in a single batch.
Generation  | Use record generation to verify that a record didn't change since the last read.
Expire  | Set an expiration on records.
Touch | Extend the life of records about to expire.
Operate | Perform multiple operations on a single record in one database command.
Delete Bin  | Delete a bin in a record.
Scan Parallel | Scan all records in a namespace and set in parallel using one thread-per-server node.
Scan Series | Scan all records in a namespace and set by querying each server node in series.
List/Map  | Write and read records containing combinations of List and Dictionary bins.
User Defined Function | Call UDFs on the server.

### Usage

Either choose to run one example or run all examples in one sweep:

```bash
go get github.com/aerospike/aerospike-client-go
cd $GOPATH/src/github.com/aerospike/aerospike-client-go/examples
go run <name> -h <host> -p <port> -n <namespace> -s <set>
```

To run the Touch example on namespace _test_ within set _demoset_:

```bash
go run examples/touch.go -h localhost -p 3000 -n test -s demoset
```
