---
title: Asynchronous API
description: ase the Aerospike C# client library asynchronous API to place commands on a queue and return control to the application. 
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Use the Aerospike C# client library asynchronous API (the `AsyncClient` class) to place commands on a queue and return control to the application, which allows an executor to  asynchronously process the commands.

The `AsyncClient` instance is thread-safe and can concurrent. A separate thread uses a pool of non-blocking sockets to send the command, process the reply, and notify the caller with the result. `AsyncClient` uses less threads and makes efficient use of threads than using the synchronous client. One con of using `AsyncClient` is that the programming model is difficult to implement, debug, and maintain.

### Example

These examples demonstrate the following standard operations:

- Connecting to the Aerospike cluster.
- Writing a single value.
- Reading a single value.
- Making the application wait.


```cs
using System;
using System.Threading;
using Aerospike.Client;
 
namespace Test
{
    public class AsyncTest
    {
        private AsyncClient client;
        private WritePolicy policy;
        private bool completed;
 
        public AsyncTest()
        {
            policy = new WritePolicy();
            policy.timeout = 50; // 50 millisecond timeout.
        }
 
        public void RunTest()
        {
            client = new AsyncClient("127.0.0.1", 3000);
            try
            {
                // Write a single value.  
                Key key = new Key("test", "myset", "mykey");
                Bin bin = new Bin("mybin", "myvalue");
                Console.WriteLine(string.Format("Write: namespace={0} set={1} key={2} value={3}", 
                key.ns, key.setName, key.userKey, bin.value));
                client.Put(policy, new WriteHandler(this, key, bin), key, bin);
                WaitTillComplete();
            }
            finally
            {
                client.Close();
            }
        }
 
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
 
        private void WaitTillComplete()
        {
            lock (this)
            {
                while (!completed)
                {
                    Monitor.Wait(this);
                }
            }
        }
 
        private void NotifyCompleted()
        {
            lock (this)
            {
                completed = true;
                Monitor.Pulse(this);
            }
        }
    }
}
```


### Application Wait

To make the application wait and avoid closing the connection before the write and read commands complete:

```cs
private void WaitTillComplete()
{
    lock (this)
    {
        while (!completed)
        {
            Monitor.Wait(this);
        }
    }
}
 
private void NotifyCompleted()
{
    lock (this)
    {
        completed = true;
        Monitor.Pulse(this);
    }
}
```

