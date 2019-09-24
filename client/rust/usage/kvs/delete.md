---
title: Delete a Record
description: Use the Aerospike Rust client to delete records from the Aerospike database.
---

This example deletes the key _mykey_ from the namespace _test_ in the set _myset_:

```rust
let key = as_key!("test", "test", "mykey");
match client.delete(&WritePolicy::default(), &key) {
    Ok(true) => println!("Record deleted"),
    Ok(false) => println!("Record did not exist"),
    Err(err) => println!("Error deleting record: {}", err),
}
```
