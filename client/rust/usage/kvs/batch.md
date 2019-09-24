---
title: Batch Reads
description: Use the Aerospike Rust client to read multiple records in one sweep from the Aerospike Database.
---

Multiple records can be read in a single batch call.

```rust
let bins = ["name", "age"];
let mut batch_reads = vec![];
for i in 0..10 {
  let key = as_key!("test", "test", i);
  batch_reads.push(BatchRead::new(key, &bins));
}
match client.batch_get(&BatchPolicy::default(), batch_reads) {
    Ok(results) => {
      for result in results {
        match result.record {
          Some(record) => println!("{:?} => {:?}", result.key, record.bins),
          None => println!("No such record: {:?}", result.key),
        }
      }
    }
    Err(err) => println!("Error executing batch request: {}", err),
}
```

This call groups keys based on which Aerospike Server node can best handle the
request, and uses the client's thread pool to concurrently handle all requests
to each node. After all nodes return the record data, the records are returned
to the caller. The list of records returned is in the same order the keys are
passed in. If a record is not found in the database, the batch read's record
entry is `None`.
