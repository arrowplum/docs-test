---
title: Rust Client Best Practices
description: Follow these best practices when using the Aerospike Rust client API. 
categories:
  - aerospike-client-rust
tags:
  - aerospike-client-rust
  - Rust
---

Follow these best practices when using the Aerospike Rust client API.

- Each `Client` instance spawns a maintenance thread that periodically makes
  info requests to all server nodes for partition mapping. Multiple client
  instances create additional load on the server. Use only one client instance
  per cluster in an application and share that instance among multiple threads
  using the `std::sync::Arc` smart pointer.

- By default, the user-defined key is not stored on the server. It is converted
  to a hash digest which is used to identify a record. If the user-defined key
  must persist on the server, use one of the following methods:
    - **Set `WritePolicy.send_key` to true** &mdash; the key is sent to the
      server for storage on writes, and retrieved on multi-record scans and
      queries.
    - Explicitly store and retrieve the user-defined key in a bin.

- Use `Client.operate()` to batch multiple operations (add/get) on the same
  record in a single call.

- In cases where all record bins are created or updated in a transaction,
  enable Replace mode on the transaction to increase performance. The server
  then does not have to read the old record before updating. Do not use Replace
  mode when updating a subset of bins.

```rust
let mod policy = WritePolicy::default();
policy.record_exists_action = RecordExistsAction::Replace;
client.put(&policy, &key, bins).unwrap();
```

- Each database command takes in a policy as the first argument. If the policy
  is identical for a group of commands, reuse them instead of instantiating
  policies for each command.
