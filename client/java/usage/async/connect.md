---
title: Asynchronous Connect
description: Asynchronous Connect
---

The AerospikeClient instance can now be created with the assigned event loops.

```java
ClientPolicy clientPolicy = new ClientPolicy();
clientPolicy.eventLoops = eventLoops;

AerospikeClient client = new AerospikeClient(clientPolicy, new Host("host1", 3000), new Host("host2", 3000), new Host("host3", 3000));
```

AerospikeClient only needs one host to seed the cluster, but using all known hosts for seeds is still recommended because some of the hosts may be inactive.  The AerospikeClient constructor also accepts a host array.

```java
Host[] hosts = new Host[] {new Host("host1", 3000), new Host("host2", 3000), new Host("host3", 3000)};

AerospikeClient client = new AerospikeClient(clientPolicy, hosts);
```

AerospikeClient can perform both synchronous and asynchronous commands in a single instance.  EventLoops are required when performing asynchronous commands. AerospikeClient is thread-safe and can be used concurrently.

### Close

Both AerospikeClient and EventLoops should be closed before program shutdown.

```java
client.close()
eventLoops.close();
```

AerospikeClient will wait till pending asynchronous commands complete before closing.  Asynchronous commands issued after close() will be rejected.

[Next](/docs/client/java/usage/async/write.html)
