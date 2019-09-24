---
title: Connecting
description: Use the Aerospike C# client to connect and periodically ping nodes for cluster status.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

To establish an Aerospike server connection, create an `AerospikeClient` object by providing the IP address and port of one cluster node. This IP address is used to initiate contact to the cluster, and the client can then discover the entire cluster.

```cs
AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);
```

The `AerospikeClient` constructor creates a maintenance thread that periodically pings nodes for cluster status.

On a network disturbance where the client can no longer reach any nodes, the seed nodes (and the discovered friend nodes on initial connection) are pinged until client-server connection is reestablished.

The `AerospikeClient` instance is thread-safe and can be used concurrently. Each `get/set` call is a blocking, synchronous network call to Aerospike. Connections are cached with a connection pool for each server node.

## Cleaning Up

Call `Close()` when all transactions are finished and the application is ready to shutdown. 

{{#note}}
The `AerospikeClient` object can no longer be called after calling `Close)`.
{{/note}}

```cs
client.Close();
```
