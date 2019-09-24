---
title: Introduction - Java Client
description: Aerospike’s Java client allows you to build applications in Java that store and retrieve data from an Aerospike cluster.
categories:
  - aerospike-client-java
tags:
  - aerospike-client-java
  - Java
---

The Aerospike Java client enables you to build Java applications to store and retrieve data from an Aerospike cluster. It contains both synchronous and asynchronous calls to the database.

The Aerospike Java client runs on any platform with Java JDK 8 and above.


### Code Example

```java
AerospikeClient client = new AerospikeClient("192.168.1.150", 3000);

Key key = new Key("test", "demo", "putgetkey");
Bin bin1 = new Bin("bin1", "value1");
Bin bin2 = new Bin("bin2", "value2");

// Write a record
client.put(null, key, bin1, bin2);

// Read a record
Record record = client.get(null, key);

client.close();
```

<div class="text-center">
<a class="button primary" href="/docs/client/java/start">GET STARTED</a>
</div>
