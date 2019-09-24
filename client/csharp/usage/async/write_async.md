---
title: Asynchronous Write
description: Use the Aerospike C# Client APIs to asynchronously write to the Aerospike database.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Use the Aerospike C# Client APIs to asynchronously write to the Aerospike database.

### Write a Single Value

The `AsyncClient` method takes an extra `WriteHandler` listener parameter that notifies the caller on command success or failure:

```cs
// Write a single value.
Key key = new Key("test", "myset", "mykey");
Bin bin = new Bin("mybin", "myvalue");
client.Put(policy, new WriteHandler(this, key, bin), key, bin);
```

For the read to wait until the write completes, the `RecordHandler` read initiates within the`WriteHandler` listener `onSuccess()` method.

```cs
private class WriteHandler : WriteListener
{
    private readonly AsyncTest parent;
    private readonly Key key;
    private readonly Bin bin;

    public WriteHandler(AsyncTest parent, Key key, Bin bin)
    {
        this.parent = parent;
        this.key = key;
        this.bin = bin;
    }

    public void OnSuccess(Key key)
    {
        try
        {
            // Write succeeded.  Now call read.
            parent.client.Get(parent.policy, new RecordHandler(parent, key), key);
        }
        catch (Exception e)
        {
            Console.WriteLine(string.Format("Failed to get: namespace={0} set={1} key={2} exception={3}",
            key.ns, key.setName, key.userKey, e.Message));
        }
    }

    public void OnFailure(AerospikeException e)
    {
        Console.WriteLine("Failed to put: namespace={0} set={1} key={2} exception={3}",
        key.ns, key.setName, key.userKey, e.Message);
        parent.NotifyCompleted();
    }
}
 
```


