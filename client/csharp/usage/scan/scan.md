---
title: Scan Records
description: Use the Aerospike C# client APIs to scan all records in a specified namespace and set.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Use the Aerospike C# client APIs to scan all records in a specified namespace and set.

### Example

This example counts the number of records returned:

```cs
using System;
using System.Collections.Generic;
using System.IO;
using Aerospike.Client;
  
namespace Test
{
    public class ScanParallelTest
    {
        private int recordCount;
 
        public void RunTest()
        {
            AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);
 
            try
            {
                ScanPolicy policy = new ScanPolicy();
                policy.concurrentNodes = true;
                policy.priority = Priority.LOW;
                policy.includeBinData = false;
  
                client.ScanAll(policy, "test", "demoset", ScanCallback);
                Console.WriteLine("Records " + recordCount);
            }
            finally
            {
                client.Close();
            }
        }
 
        public void ScanCallback(Key key, Record record)
        {
            recordCount++;
            if ((recordCount % 10000) == 0)
            {
                Console.WriteLine("Records " + recordCount);
            }
        }
    }
}
```

The scan policy is initialized, and:
- If `concurrentNodes = true`, the scan queries nodes for records in parallel using threads. Otherwise, the scan sequentially queries each node one at a time. 
- If `includeBinData = true`, the specified bins (all bins by default) return with each record.

```cs
ScanPolicy policy = new ScanPolicy();
policy.concurrentNodes = true;
policy.priority = Priority.LOW;
policy.includeBinData = false;
```

The scan can now execute. If an exception is thrown, parallel scan threads to other nodes are also terminated, and the exception propagates back through the initiating scan call:

```cs
client.ScanAll(policy, "test", "demoset", ScanCallback);
```

The scan returns records to the specified callback. 

Note that multiple threads are likely calling the scan callback in parallel. Therefore, the scan callback implementation must be thread safe. 

This example callback counts the number of records returned:

```cs
public void ScanCallback(Key key, Record record)
{
    recordCount++;
    if ((recordCount % 10000) == 0)
    {
        Console.WriteLine("Records " + recordCount);
    }
}
```

