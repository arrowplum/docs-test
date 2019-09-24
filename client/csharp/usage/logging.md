---
title: Logging
description: Use the Aerospike C# client logging interface for debugging and informational purposes.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Use the Aerospike C# client logging interface for debugging and informational purposes (for example, the addition or deletion of server nodes). By default, logging is disabled. Provide a log callback to enable the log.

### Example

This example uses the logging interface:

```cs
using System;
using System.Collections.Generic;
using System.IO;
using Aerospike.Client;
 
namespace Test
{
    public class LogTest
    {
        public static void RunTest()
        {
            Log.SetLevel(Log.Level.INFO);
            Log.SetCallback(LogCallback);
 
            AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);
 
            try
            {
                // Perform database operations.
            }
            finally
            {
                client.Close();
            }
        }
 
        public static void LogCallback(Log.Level level, string message)
        {
            Console.WriteLine(DateTime.Now.ToString() + ' ' + level + ' ' + message); 
        }
    }
}
```

{{#note}}
 Only execute this code once at application launch.
{{/note}}
