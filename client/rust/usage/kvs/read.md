---
title: Read a Record
description: Use Aerospike Rust client APIs to read records from the Aerospike Database.
---

To read a record from the database, the Aerospike Rust client library can:

- Read all bins in a record.
- Read specified bins in a record.
- Read only the metadata of a record.

Each method returns a `Record` struct that contains the metadata and bins of the record.

### Read All Bins in a Record

This example reads all bins for a record:

```rust
let key = as_key!("test", "myset", "mykey");
match client.get(&ReadPolicy::default(), &key, Bins::All) {
    Ok(record)
        => println!("name={:?}", record.bins.get("name")),
    Err(Error(ErrorKind::ServerError(ResultCode::KeyNotFoundError), _))
        => println!("No such record: {}", key),
    Err(err)
        => println!("Error fetching record: {}", err),
}
```

### Read Specific Bins of a Record

This example only reads two bins from the record: _name_ and _age_:

```rust
let key = as_key!("test", "test", "mykey");
match client.get(&ReadPolicy::default(), &key, &["name", "age"]) {
    Ok(record)
        => println!("name={:?}", record.bins.get("name")),
    Err(Error(ErrorKind::ServerError(ResultCode::KeyNotFoundError), _))
        => println!("No such record: {}", key),
    Err(err)
        => println!("Error fetching record: {}", err),
}
```

### Read Record Metadata

Use `Bins::None` to fetch only the record's metadata - here for
example to retrieve the record's remaining time-to-live:

```rust
let key = as_key!("test", "myset", "mykey");
match client.get(&ReadPolicy::default(), &key, Bins::None) {
    Ok(record) => {
        match record.time_to_live() {
            None => println!("record never expires"),
            Some(duration) => println!("ttl: {} secs", duration.as_secs()),
        }
    },
    Err(Error(ErrorKind::ServerError(ResultCode::KeyNotFoundError), _))
        => println!("No such record: {}", key),
    Err(err)
        => println!("Error fetching record: {}", err),
}
```
