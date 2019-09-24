---
title: Go Client Usage
description: Use the Aerospike Go client to build Go applications to store and retrieve data in the Aerospike database.
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
  - Go
---

Use the Aerospike Go client to build Go applications to store and retrieve data in the Aerospike database.

The Go client runs on any Go v1.2+ platform .

### Code

This example establishes a connection to the Aerospike server, configures a key, writes and reads a record, ands closes the server connection.

```go
client, err := as.NewClient("192.168.1.150", 3000)

key, err := as.NewKey("namespace", "set", 
	"key value goes here and can be any supported primitive")

bin1 := as.NewBin("bin1", "value1")
bin2 := as.NewBin("bin2", "value2")

// Write a record
err = client.PutBins(nil, key, bin1, bin2)

// Read a record
record, err := client.Get(nil, key)

client.Close()
```

<div class="text-center">
<a class="button primary" href="/docs/client/go/start">GET STARTED</a>
</div>
