---
title: Scan Records
description: The Aerospike Rust client provides the ability to scan all records in a specified Namespace and set. 
---

The Aerospike Rust client provides the ability to scan all records in a
specified Namespace and set. 

### Scanning Records

This example counts the number of records in the scan result:

```rust
#[macro_use]
extern crate aerospike;

use aerospike::*;

fn main() {
    let hosts = &std::env::var("AEROSPIKE_HOSTS").unwrap();
    let client = Client::new(&ClientPolicy::default(), &hosts).unwrap();
    let mut policy = ScanPolicy::default();
    policy.include_bin_data = false;
    match client.scan(&policy, "test", "demo", None) {
        Ok(records) => {
            let mut count = 0;
            for record in &*records {
                match record {
                    Ok(record) => count += 1,
                    Err(err) => panic!("Error executing scan: {}", err),
                }
            }
            println!("Records: {}", count);
        },
        Err(err) => println!("Error fetching record: {}", err),
    }
}
```

The scan can now execute. If an error occurs while executing the scan, parallel
scan threads to other nodes are terminated and the error propagates back
through the record scan.
