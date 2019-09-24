---
title: Scan Records
description: Use the Aerospike Go client to scan all records in the specified namespace and set.
---

Use the Aerospike Go client to scan all records in the specified namespace and set. 

### Running the Scan

To count the number of records returned:

```go
import (
    as "github.com/aerospike/aerospike-client-go"
)

func main() {
    client, err := as.NewClient("127.0.0.1", 3000)
    // deal with the error here

    defer client.Close()
     
    spolicy := as.NewScanPolicy()
    spolicy.ConcurrentNodes = true
    spolicy.Priority = as.LOW
    spolicy.IncludeBinData = false

    recs, err := client.ScanAll(spolicy, "test", "demoset")
    // deal with the error here

    for res := range recs.Results() {
      if res.Err != nil {
        // handle error here
        // if you want to exit, cancel the recordset to release the resources
      } else {
        // process record here
        fmt.Println(res.Record.Bins)
      }
    }
}
```

First, the scan policy is initialized.

- `ConcurrentNodes = true` &mdash; The scan queries nodes for records in parallel using goroutines. Otherwise, the scan queries each node sequentially. 
-`IncludeBinData = true`&mdash; Returns the specified bins with each record (default = all bins) .

```go
spolicy := as.NewScanPolicy()

spolicy.ConcurrentNodes = true
spolicy.Priority = as.LOW
spolicy.IncludeBinData = false
```

Execute the scan. 

Errors stop the scan on that node, and return on the `Recordset.Errors` channel.
