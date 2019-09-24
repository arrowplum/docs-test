---
title: Aynchronous Connect
description: Use the Aerospoke C# client to establish connection to Aerospike server by creating an `AsyncClient` object and providing it the IP address and port of the node or one of the nodes in a cluster.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Use the Aerospoke C# client to establish connection to Aerospike server by creating an `AsyncClient` object and providing it the IP address and port of one node in a cluster. In a cluster, this IP address is used to initiate contact to the cluster, and then the client can discover all cluster nodes and cluster status.

```
AsyncClient client = new AsyncClient("127.0.0.1", 3000);
```

The `AsyncClient` constructor creates a maintenance thread to periodically ping nodes for cluster status.

On a network disturbance where the client can no longer reach any node, the seed nodes (and the discovered friend nodes on initial connection) are pinged until client-server connection is reestablished.

The `AsyncClient` instance is thread-safe and can be concurrent. Each `get/set` call is a blocking, synchronous network call to Aerospike. Non-blocking connections are cached in a connection pool for each server node.
