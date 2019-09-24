---
title: Introduction - C# Client
description: Use the Aerospike C# client to build synchronous and asynchronous C# applications to store and retrieve data from an Aerospike cluster.
---

The Aerospike C# client enables you to build C# applications to store and retrieve data from an Aerospike cluster. Both .NET Framework and .NET Core target platforms are supported.

### Code Example

```
// Establish connection the server
AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);

// Create key
Key key = new Key("test", "myset", "mykey");

// Create Bins
Bin bin1 = new Bin("name", "John");
Bin bin2 = new Bin("age", 25);

// Write record
client.Put(null, key, bin1, bin2);

// Read record
Record record = client.Get(null, key);

// Close connection
client.Close();
```

<div class="text-center">
<a class="button primary" href="/docs/client/csharp/start">GET STARTED</a>
</div>
