---
title: Go Client Best Practices
description: Best practices for using the Aerospike Go client API with the Aerospike database. 
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
  - go
---

These best practices are for using the Aerospike Go client API with the Aerospike database.

### Optimizing Performance

The Aerospike Go client library provides tools to optimize performance for different workloads. For the best possible performance, ensure that you benchmark and profile your application under production workloads.

#### Client Design Goals

Using the Aerospike Go client library to optimize application performance achieves these goals:

- **Minimal Memory Allocation**: Aerospike removes as many allocations as possible by pooling buffers and hash objects whenever possible.
- **Customization Friendly**: Use the Aerospike Go client library parameters to customize variables to influence performance under different workloads.
- **Determinism**: Aerospike keeps client inner workings deterministic, and stays away from data structures or algorithms that are not deterministic. All pool and queue implementations in the client are bound in maximum memory size and perform in predetermined number of cycles. There are no heuristic algorithms in the client.

#### Best Practices

Follow these recommendations to optimize your application.

1. **Limit server connections** &mdash; Each server node has a limited number of file descriptors on the operating system for connections. No matter how big, this resource is limited and can be exhausted by too many connections. Clients pool their connections to database nodes for optimal performance. If node connections are exhausted by existing clients, new clients cannot connect to the database (for example, when launching a new application in the cluster).
  To guard against this, design your application design with the following:
  1. **Use only one `Client` object in your application** &mdash; `Client` objects pool connections and synchronize their inner functionality. They are goroutine friendly. Use only one `Client` object in your application and pass it around.
  1. **Limit `Client` connection pool** &mdash; The default number of maximum connection pools in a `Client` object is 256. Even under extreme load in fast metal, clients rarely use more than quarter of these connections. When no connections are available in the pool, the client opens a new connection to server. If all connections in the pool are open, connections are closed on operation completion to prevent too many connections.
  If the pool is too small, the client wastes time connecting to the server for each new request. If the pool is too large, it wastes server connections.
  At its maximum number of 256 for each client and with `proto-fd-max` set to 10000 in your server node configuration, 50 clients **per server node** is safe practice. This will approach 150 high-performing clients. Change the pool size in `ClientPolicy`, and then initialize your `Client` object using the `NewClientWithPolicy(policy *ClientPolicy, hostname string, port int)` initializer.
1. **Pool client buffers** &mdash; The Aerospike client library pools buffers to reduce memory allocation. Note that unbounded memory pools are bugs you haven't found yet, this pool implementation enforces these bounds on pool:
  1. Don't increase buffer sizes. Initial buffer sizes are large enough for most operations (default is 16KiBs).
  1. Pool size is limited (default is 512 buffers).
  1. Buffer sizes are limited. If a buffer is larger than the limit, it is not returned to the pool. (default is 128 KiBs).
  These limits ensure that the pool is deterministic in performance and memory size (default is a maximum of 64MiB). While the default values perform well under most circumstances, the following conditions could impact performance:
  - Initial buffer size too small and final size too large &mdash; Each allocation will be too small for the operation (for example, with records larger than 128KiB).
  - Pool size too small and initial buffer size too small or final buffer size too large &mdash; Buffers are allocated and then dismissed because of average pool size.
  If pool size is a bottleneck in your application, change it using `SetCommandBufferPool(poolSize, initBufSize, maxBufferSize int)`. Note that the pool is a package object shared between all clients. 
3. **Use `Bin` objects in `Put` operations instead of `BinMap`** &mdash; `Put` methods require passing a map for bin values. This allocates an array of bins on each call, iterates on the map, and creates `Bin` objects.
  If performance is absolutely crucial, avoid `BinMap` allocation by using `PutBins` to manually pass the bins, allocate `[]Bin`, and iterate over the `BinMap` (this creates two allocations and an O(n) algorithm).

