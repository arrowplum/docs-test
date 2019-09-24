---
title: Installation
description: Use the Aerospike Go client to build Go applications to store and retrieve data from an Aerospike cluster.
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
  - install
---

This describes installing the Aerospike Go client.

### Prerequisites

- [Go](http://golang.org) version v1.2+.

{{#note}}
You can build the code in Go versions prior to 1.2, but our test library relies on v1.2.
{{/note}}

To install the latest stable version of Go, see [http://golang.org/dl/](http://golang.org/dl/). Your package manager may also have `go`. To see if `go` is installed and working, try `go version`.

The Aerospike Go client implements the Aerospike Aerospike network protocols in pure Go in pure Go, and does not depend on the C client. It is goroutine friendly, and works asynchronously.

Supported operating systems:

- Major Linux distributions (Ubuntu, Debian, Redhat)
- Mac OS X
- Windows (untested)

### Installing

1. Install Go v1.2+ and setup your environment as documented [here](http://golang.org/doc/code.html#GOPATH). Your package manager may also have `go`. To see if `go` is installed and working, try `go version`.
2. Get the client in your `$GOPATH`:

```bash
go get github.com/aerospike/aerospike-client-go
```

3. Update the client library:

```bash
go get -u github.com/aerospike/aerospike-client-go
```

### Testing

The Aerospike Go client package contains a number of tests.

{{#note}}
Tests require Ginkgo and Gomega library.
```bash
go get github.com/onsi/ginkgo
go get github.com/onsi/gomega
```
{{/note}}

Before running the tests, update dependencies:

```bash
cd $GOPATH/src/github.com/aerospike/aerospike-client-go/tools/benchmark
go get .
```

To run all test cases with race detection:

```bash
ginkgo -r -race -- -h 127.0.0.1 -p 3000
```

### Next Steps
- Run a [Read or Write](/docs/client/go/examples.html)
- Try the [Benchmark Tool](/docs/client/go/benchmarks.html)
