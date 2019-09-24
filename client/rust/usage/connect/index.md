---
title: Connecting
description: The Aerospike client object can connect and periodically ping nodes for the current state of the cluster. 
---

To connect to an Aerospike database, create a new `Client` instance that
specifies the IP address and port of one or more seed nodes in the cluster.

## Single Seed Node

The client first connects to a seed node, and then discovers the rest of the cluster.

```rust
extern crate aerospike;

use aerospike::{Client, ClientPolicy};

let client = Client::new(&ClientPolicy::default(), "127.0.0.1:3000").unwrap();
```

The port number can be omitted; the client will default to port number 3000.

## Multiple Seed Nodes

Multiple seed nodes can also be provided. The client iterates through the array
of nodes until it successfully connects to a node. It then discovers all nodes
in the cluster.

```rust
extern crate aerospike;

use aerospike::{Client, ClientPolicy};

let client = Client::new(&ClientPolicy::default(), "10.0.10.1,10.0.10.2,10.0.10.3").unwrap();
```

## Maintenance Thread

The AerospikeClient constructor creates a maintenance thread that periodically
pings nodes for cluster status. If a network disturbance is detected and the
client can‘t reach any nodes, the seed nodes are used until client-server
connection is reestablished.

The Aerospike Client instance is thread-safe and can be used concurrently. Each
get/set call is a blocking, synchronous network call to Aerospike. Connections
are cached with a connection pool for each server node.

# Cleanup

When all transactions complete and the application is prepared for a clean
shutdown, call the `close()` method to remove resources held by the
Client instance. 

```rust
client.close().unwrap();
```
{{#note}}
The `Client` instance cannot be used after calling `close()`.
{{/note}}
