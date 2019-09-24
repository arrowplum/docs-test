---
title: Asynchronous Read
description: Use the Aerospike C# client APIs to asynchronously read a single value in the Aerospike database.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Use the Aerospike C# client APIs to asynchronously read a single value in the Aerospike database.

### Read a Single Value

The `RecordHandler` single record listener is notified when the read completes and a record returns. The parent is then notified that both write and read operation successfully completed.

```cs
private class RecordHandler : RecordListener
{
    private readonly AsyncTest parent;
    private readonly Key key;
 
    public RecordHandler(AsyncTest parent, Key key)
    {
        this.parent = parent;
        this.key = key;
    }
 
    public void OnSuccess(Key key, Record record)
    {
        // Read completed.
        object received = (record == null) ? null : record.GetValue("mybin");
        Console.WriteLine(string.Format("Received: namespace={0} set={1} key={2} value={3}",
        key.ns, key.setName, key.userKey, received));
        // Notify application that read is complete.
        parent.NotifyCompleted();
    }
 
    public void OnFailure(AerospikeException e)
    {
        Console.WriteLine("Failed to get: namespace={0} set={1} key={2} exception={3}",
        key.ns, key.setName, key.userKey, e.Message);
        parent.NotifyCompleted();
    }
}
```
