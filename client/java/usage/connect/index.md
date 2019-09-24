---
title: Connecting
description: The Aerospike client object can connect and periodically ping nodes for the current state of the cluster. 
---

To connect to an Aerospike database, create an `AerospikeClient` object that specifies the IP address and port of one or more seed nodes in the cluster.

## Single Seed Node

The client first connects to a seed node, and then discovers the rest of the cluster.

```java
import com.aerospike.client.AerospikeClient;

AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);
```

## Multiple Seed Nodes

Multiple seed nodes can also be provided.  The client iterates through the array of nodes until it successfully connects to a node. It then discovers all nodes in the cluster.

```java
import com.aerospike.client.AerospikeClient;
import com.aerospike.client.Host;
import com.aerospike.client.policy.ClientPolicy;

Host[] hosts = new Host[] {
    new Host("a.host", 3000),
    new Host("another.host", 3000),
    new Host("and.another.host", 3000)
};

AerospikeClient client = new AerospikeClient(new ClientPolicy(), hosts);
```

## User Authentication

Aerospike enterprise servers may require user authentication.

```java
import com.aerospike.client.AerospikeClient;
import com.aerospike.client.Host;
import com.aerospike.client.policy.ClientPolicy;

Host[] hosts = new Host[] {
    new Host("a.host", 3000),
    new Host("another.host", 3000),
    new Host("and.another.host", 3000)
};

ClientPolicy policy = new ClientPolicy();
policy.user = "myuser";
policy.password = "mypass";

AerospikeClient client = new AerospikeClient(policy, hosts);
```

## Maintenance Thread

The AerospikeClient constructor creates a maintenance thread that periodically pings nodes for cluster status. If a network disturbance is detected and the client can‘t reach any nodes, the seed nodes are used until client-server connection is reestablished.

The AerospikeClient instance is thread-safe and can be used concurrently. Connections are cached with connection pool(s) for each server node.

# Cleanup

When all transactions complete and the application is prepared for a clean shutdown, call the `close()` method to remove resources held by the AerospikeClient object. 

```java
client.close();
```
{{#note}}
The `AerospikeClient` object cannot be used after calling `close()`.
{{/note}}

