---
title: Ruby Client Best Practices
description: Best practices for using the Aerospike Ruby client with the Aerospike database. 
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
  - ruby
---

Follow these best practices for using the Aerospike Ruby client Aerospike database.

### Tweaking Performance

Use these practices to tweak performance for different workloads. 

{{#note}}
To achieve the best possible performance, benchmark and profile your application using production workloads.
{{/note}}

#### Client Design Goals

The Aerospike Ruby client library was placed under various workloads to achieve the following goals:

- **Minimal memory allocation**: The Aerospike Ruby client has as few allocations as possible. Buffers and hash objects are pooled whenever possible to achieve this goal.
- **Customization friendly**: Use parameters to customize variables that influence performance under different workloads.
- **Determinism**: The Aerospike Ruby client is deterministic. Data structures and algorithms that are not deterministic are avoided. All client pool and queue implementations are bound in maximum memory size and perform a determined number of cycles. There are no heuristic algorithms in the client.

#### Optimizing Performance

Follow these best practices to optimize system performance:

1. **Server Connection Limit**: Each server node has a limited number of connection file descriptors on the operating system. No matter how big, this resource is limited and can be exhausted by too many connections. Clients pool connections to database nodes for optimal performance. If node connections are exhausted by existing clients, new clients cannot connect to the database (for example, when a new application launches in the cluster).

  To minimize cluster connections, observe the following when designing your application:

  - **Call only one Client object**: `Client` objects pool connections and synchronize functionality. They are thread-safe. Pass the `Client` object to reserve cluster connections.
  - **Limit `Client` connection pools**: The maximum `Client` object connection pool size is 64. Even under extreme load in fast metal, clients rarely use more than 1/4 of these connections. If no connections are available in the pool, a new connection to server is created. If the pool is full, connections are closed after use to minimize used connections. If this pool is too small, the client wastes time connecting to the server on each request. If the pool is too big, it wastes server connections.

  With the maximum 64 connections for each client and `proto-fd-max` set to 10000 in the server node configuration, there can safely be approximately **150 clients per server node**. In practice, this can easily approach 600 high-performing clients. Set the connection queue size in `ClientPolicy` when initializing the client.

2. **Client Buffer Pool**: The Aerospike Ruby client pools its buffers to reduce memory allocation and enforces two bounds on the pool:

  - Initial buffer sizes are large enough (16 KiBs by default) for common operations and data sizes. It may be unnecessary to increase buffer size. 
  - Pool size is limited (32 buffers by default).
  - Buffer size is limited (10 MiBs by default). If a buffer is bigger than this limit, it isn't placed in the pool. 

  These limits make the buffer pool deterministic in performance and memory size (maximum of 320MiB by default, but is much less in practice).
  
