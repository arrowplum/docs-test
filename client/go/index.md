---
title: Introduction - Go Client
description: Use the Aerospike Go client to build Go applications to store and retrieve data in the Aerospike database.
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
  - Go
---

Use the Aerospike Go client to build  Go applications to store and retrieve data from an Aerospike cluster.

The Aerospike Go client runs on any platform with Go 1.2 version and above.

### Code

This example establishes a connection to the Aerospike server, configures a key, writes and reads a record, ands closes the server connection.


```go
client, err := as.NewClient("192.168.1.150", 3000)
if err != nil {
	log.Fatal(err)
}

key, err := as.NewKey("namespace", "set",
	"key value goes here and can be any supported primitive")
if err != nil {
	log.Fatal(err)
}

bin1 := as.NewBin("bin1", "value1")
bin2 := as.NewBin("bin2", "value2")

// Write a record
err = client.PutBins(nil, key, bin1, bin2)
if err != nil {
	log.Fatal(err)
}

// Read a record
record, err := client.Get(nil, key)
if err != nil {
	log.Fatal(err)
}

client.Close()
```

<div class="text-center">
<a class="button primary" href="/docs/client/go/start">GET STARTED</a>
</div>
