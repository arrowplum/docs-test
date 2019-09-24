---
title: Query Records
description: Use the Aerospike C# client to query on secondary indexes.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

In addition to querying using the primary index using [key-value store](/docs/client/csharp/usage/kvs/index.html) API, the Aerospike C# client provides an API to query the database on secondary indexes.

You can run Equality and Range queries on numeric indexes. For string indexes, only Equality queries are available. These queries return records matching the query criteria.

See [Manage Indexes](/docs/client/csharp/usage/query/sindex.html) for information on secondary indexes.

To continue, the client must have an [established the cluster connection](/docs/client/csharp/usage/connect_sync.html).

### Example

This example demonstrates:

- Creating a secondary index on an integer type bin.
- Writing records to that bin.
- Executing a query using the Range filter.

```cs
using System;
using System.Collections.Generic;
using System.IO;
using Aerospike.Client;
 
namespace Test
{
    public class QueryTest
    {
        public static void RunTest()
        {
            AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);
 
            try
            {
                string ns = "test";
                string set = "demoset";
                string indexName = "queryindexint";
                string keyPrefix = "querykeyint";
                string binName = "querybinint";
                int size = 50;
 
                CreateIndex(client, ns, set, indexName, binName);
                WriteRecords(client, ns, set, keyPrefix, binName, size);
                RunQuery(client, ns, set, indexName, binName);
            }
            finally
            {
                client.Close();
            }
        }
 
        private static void CreateIndex(AerospikeClient client, string ns, string set, string indexName, string binName)
        {
            Console.WriteLine("Create index");
            Policy policy = new Policy();
            policy.timeout = 0; // Do not timeout on index create.
            IndexTask task = client.CreateIndex(policy, ns, set, indexName, binName, IndexType.NUMERIC);
            task.Wait();
        }
 
        private static void WriteRecords(AerospikeClient client, string ns, string set, string keyPrefix, string binName, int size)
        {
            Console.WriteLine("Write " + size + " records.");
            WritePolicy policy = new WritePolicy();
            for (int i = 1; i <= size; i++)
            {
                Key key = new Key(ns, set, keyPrefix + i);
                Bin bin = new Bin(binName, i);
                client.Put(policy, key, bin);
            }
        }
 
        private static void RunQuery(AerospikeClient client, string ns, string set, string indexName, string binName)
        {
            Console.WriteLine("Query");
            Statement stmt = new Statement();
            stmt.SetNamespace(ns);
            stmt.SetSetName(set);
            stmt.SetBinNames(binName);
            stmt.SetFilters(Filter.Range(binName, 14, 18));
 
            RecordSet rs = client.Query(null, stmt);
 
            try
            {
                while (rs.Next())
                {
                    Key key = rs.Key;
                    Record record = rs.Record;
                    object result = record.GetValue(binName);
                    Console.WriteLine("Record found: ns=" + key.ns + 
                        " set=" +  key.setName  + 
                        " bin=" +  binName + 
                        " digest=" + ByteUtil.BytesToHexString(key.digest) +
                        " value=" +  result);
                }
            }
            finally
            {
                rs.Close();
            }
        }
    }
}
```
 
### Understanding Query

These examples demonstrate different query operations: 

- Define a query with a Range filter:
   
  ```cs
  Statement stmt = new Statement();
  stmt.SetNamespace(ns);
  stmt.SetSetName(set);
  stmt.SetBinNames(binName);
  stmt.SetFilters(Filter.Range(binName, 14, 18));
  ```

- Execute a query and iterate through the result set:
  
  ```cs
  RecordSet rs = client.Query(null, stmt); 
  while (rs.Next())
  {
      Key key = rs.Key;
      Record record = rs.Record;
      object result = record.GetValue(binName);
      Console.WriteLine("Record found: ns=" + key.ns + 
          " set=" +  key.setName  + 
          " bin=" +  binName + 
          " digest=" + ByteUtil.BytesToHexString(key.digest) +
          " value=" +  result);
  }
  ```

