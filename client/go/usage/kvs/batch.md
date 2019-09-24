---
title: Batch Reads
description: Use the Aerospike Go client to read multiple records in sweep from the Aerospike database.
---

Use the Aerospike Go client to read multiple records from the cluster in one request. 

### Initializing Key Arrays

To set an array of keys for the application to read:

```go
keys := make([]*Key, size)

for i := 0; i < size; i++ {
	keys[i], _ = as.NewKey("test", "myset", (i + 1))
}
```

The keys must all belong to the same namespace.

### Batch Read Request

To create a batch read request:

```go
	records, err := client.BatchGet(policy, keys)
```

Internal to the Aerospike Go client, this call groups keys based on which Aerospike server node can best handle the request, and uses multiple goroutines to  concurrently handle the requests to all nodes. After all nodes have returned the record data, the records return to the caller. The array of records that returns is in the same order as the key passed in.

If a record is not found on the database, the array entry is `nil`.

