---
title: Write a Record
description: Use the Aerospike C# client to write records to the Aerospike database.
---

The Aerospike C# client library provides various forms of `AerospikeClient.Put()` to write records to the database. Note that the set and bins are  automatically created if they do not already exist. Aerospike is schemaless, so you don't have to define the schema.

To set the _mybin_ bin in the mykey record in the _test_ namespace with the `myvalue` string value:

```cs
// Initialize policy.
WritePolicy policy = new WritePolicy();
policy.timeout = 50;  // 50 millisecond timeout.
```

The write policy specifies the following for the transaction:

- Retransmission policy
- Timeout interval
- Record expiration (time-to-live)
- What to do if the record already exists. 

Instantiating the policy or passing in a NULL policy initializes the client defaults.

### Writing a Single Value

To write a single value:

```cs
// Write single value.
Key key = new Key("test", "myset", "mykey");
Bin bin = new Bin("mybin", "myvalue");
client.Put(policy, key, bin);
```

### Writing Multiple Values

To write multiple values:

```cs
// Write multiple values.
Key key = new Key("test", "myset", "mykey");
Bin bin1 = new Bin("name", "John");
Bin bin2 = new Bin("age", 25);
client.Put(policy, key, bin1, bin2);
```

