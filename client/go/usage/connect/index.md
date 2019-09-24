---
title: Connecting
description: Use the Aerospike Go client to connect and periodically ping nodes for cluster status. 
---

Use the Aerospike Go client to connect and periodically ping nodes for cluster status by creating a `Client` object to specify the IP address and port of one or more cluster seed nodes. 

### Single Seed Node

When creating a new `Client` object, specify the server to connect to using the IP address and port. The client makes initial contact with the specified server, then automatically discovers all other cluster nodes.

To create a new `Client` object:

```go
import as "github.com/aerospike/aerospike-client-go"

client, err := as.NewClient("127.0.0.1", 3000)
```

### Multiple Seed Nodes

To connect to any node in the cluster, specify each node in the cluster when creating the client. The client iterates through the array of nodes until it successfully connects to a node, then it discovers the other cluster nodes.

```go
import as "github.com/aerospike/aerospike-client-go"

hosts := []*Host {
   	as.NewHost("a.host", 3000),
    as.NewHost("another.host", 3000),
    as.NewHost("and.another.host", 3000),
}

client, err := as.NewClientWithPolicyAndHost(nil, hosts...)
```

The `NewClient` initializer creates a maintenance goroutine to periodically ping nodes for cluster status. The `Client` instance is goroutine friendly and can be used concurrently. 

Each `get/set` call is a non-blocking, asynchronous network call to the Aerospike database cluster. Connections are cached with a connection pool for each server node.

### Cleaning Up

When all transactions complete and the application is ready for a clean shutdown, call the `Close()` method to free resources held by the `Client` object. The `Client` object cannot be used after a `Close()` call.

```go
client.Close()
```
