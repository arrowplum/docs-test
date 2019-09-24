---
title: Create Event Loops
description: Create Event Loops
---

Create event loops that will process async commands.  Each EventLoops instance can be shared among multiple AerospikeClient instances.  The number of event loops for an EventLoops instance should approximate the number of cpu cores on your machine that are reserved for AerospikeClient processing.  Both Netty and direct NIO event loops are supported.

### Event Policy

Each event loop is configured with an [EventPolicy](https://github.com/aerospike/aerospike-client-java/blob/master/client/src/com/aerospike/client/async/EventPolicy.java) instance.

```java
EventPolicy eventPolicy = new EventPolicy();
```

#### Event Loop Throttle

Async commands are immediately executed on an event loop by default.  The event loop can be overloaded if the rate of commands executed on the event loop consistently exceeds the rate the event loop can process those commands.  This can cause the client to consume too many sockets which negatively affects performance.  In extreme cases, the application can run out of sockets.

These EventPolicy defaults assume the user will apply an external throttle to mitigate event loop overload.  The example [AsyncTest](/docs/client/java/usage/async/index.html) performs this throttle by seeding the event loop with N commands and running exactly one more command when a command completes.

If the user does not apply an external throttle, it's important to set `EventPolicy.maxCommandsInProcess` to limit the number of concurrent commands allowed on the event loop.  Excess commands are placed on a delay queue without an assigned socket connection until a command slot becomes available.  When a slot becomes available, the command will be assigned a socket connection and executed on the event loop.

### Create Netty Event Loops

Netty allows users to share their existing event loops with AerospikeClient which can improve performance.  Netty event loops are also required when using TLS connections.  Netty is an optional external library dependency.

#### Netty NIO

```java
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;

import com.aerospike.client.async.EventLoops;
import com.aerospike.client.async.EventPolicy;

EventPolicy eventPolicy = new EventPolicy();

// Create 4 Netty event loops using the Netty API.
EventLoopGroup group = new NioEventLoopGroup(4);

// Create Aerospike compatibile wrapper of Netty event loops.
EventLoops eventLoops = new NettyEventLoops(eventPolicy, group);
```

#### Netty EPOLL

```java
import io.netty.channel.EventLoopGroup;
import io.netty.channel.epoll.EpollEventLoopGroup;

import com.aerospike.client.async.EventLoops;
import com.aerospike.client.async.EventPolicy;

EventPolicy eventPolicy = new EventPolicy();

// Create 4 Netty event loops using the Netty API.
EventLoopGroup group = new EpollEventLoopGroup(4);

// Create Aerospike compatibile wrapper of Netty event loops.
EventLoops eventLoops = new NettyEventLoops(eventPolicy, group);
```

### Create Direct NIO Event Loops

Direct NIO event loops are an alternative implementation that is lighter weight and slightly faster than Netty defaults when not sharing event loops.  Direct NIO does not have an external library dependency.

```java
import com.aerospike.client.async.EventLoops;
import com.aerospike.client.async.EventPolicy;
import com.aerospike.client.async.NioEventLoops;

EventPolicy eventPolicy = new EventPolicy();

// Create 4 Aerospike NIO event loops directly.
EventLoops eventLoops = new NioEventLoops(eventPolicy, 4);
```

[Next](/docs/client/java/usage/async/connect.html)
