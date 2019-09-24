---
title: Read a Record
description: Use the Aerospike C# client to read records from the Aerospike database.
---

To read a record from the database, the Aerospike C# Client library has various forms of `AerospikeClient.Get()`. Each method returns a `Record` object that contains the metadata and bin(s) of a record.

### Reading a Single Value

To read a single value:

```
Record record = client.Get(policy, key, "name");
if (record != null)
{
    Console.WriteLine("Got name: " + record.GetValue("name"));
}
```

### Reading Multiple Values

To read multiple values:

```
Record record = client.Get(policy, key);
if (record != null)
{
    foreach (KeyValuePair<string, object> entry in record.bins)
    {
        Console.WriteLine("Name=" + entry.Key + " Value=" + entry.Value);
    }
}
```

