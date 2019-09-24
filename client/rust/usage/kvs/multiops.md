---
title: Multiple Ops on a Record
description: Use the Aerospike Rust client API to modify and read back to the client bins of a record in a single transaction. 
---

You can use the Rust client API to perform separate operations on multiple bins
in a record in a single transaction. This feature allows the application to
modify and read bins of a record in a single transaction (that is, perform an
atomic modification that returns the result).

Some record operations include:

Operation | Description | Conditions
--- | --- | ---
write | Write a value to a bin. |
get | Read the value of a bin. |
get_header | Read only the metadata (generation and time-to-live) of the record. |
add | Add an integer to the existing value of the bin. | The existing value must be integer.
append | Append a string to the existing value of the bin. | The existing value must be string.
prepend | Prepend a string to the existing value of the bin. | The existing value must be string.
touch | Rewrite the same record. | Generation and time-to-live are updated.

{{#note}}
Operations are performed in user defined order.
{{/note}}

### Operation Specification

To specify operations on different bins in the same transaction, create the bins with the values to apply:

```rust
let key = as_key!("test", "demoset", "opkey");
let bin1 = as_bin!("optintbin", 7);
let bin2 = as_bin!("optstringbin", "string value");	
client.put(&policy, &key, &vec![&bin1, &bin2]).unwrap();

let bin3 = as_bin!(bin1.name, 4);
let bin4 = as_bin!(bin2.name, "new string");
```

Create the appropriate operations using the bins and supply them to the `operate()` function:

```rust
let ops = vec![
    operations::add(&bin3),
    operations::put(&bin4),
    operations::get(),
];
match client.operate(&policy, &key, &ops) {
    Ok(record) => println!("optintbin: {}", record.bins.get(bin1.name).unwrap()),
    Err(err) => println!("Error writing record: {}", err),
}
```
