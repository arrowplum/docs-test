---
title: Aggregate Records
description: Use the Aerospike C# client APIs to aggregate records to process query results.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

There are various forms of processing results of a [secondary index query](/docs/client/csharp/usage/query/sindex.html). A common form is aggregation, where you apply a Stream UDF on the entire results set of a query.


Read these sections before continuing:

- **[Manage Indexes](/docs/client/csharp/usage/query/sindex.html)** &mdash; Create secondary indexes.
- **[Register UDF](/docs/client/csharp/usage/udf/register.html)** &mdash; Register a user-defined function.
- **[Query Records](/docs/client/csharp/usage/query/query.html)** &mdash; Create and execute queries.

These examples demonstrate aggregation (MapReduce) by:

- Creating the Lua package that contains the UDFs for the aggregation and copy it to the server. 
- Creating a C# program to create the index, run the query, and perform aggregation.

### Lua Aggregation

Aggregation functions run on both the server (map-and-reduce) and client (final reduce). 

To execute the `sum_example.lua` aggregation function:

```lua
local function reducer(val1,val2)
    return val1 + val2
end
 
function sum_single_bin(stream,name)
    local function mapper(rec)
        return rec[name]
    end
    return stream : map(mapper) : reduce(reducer)
end
```

### Running the Aggregation

To initialize the aggregation function, create the secondary index, and run the query: 

```cs
using System;
using System.IO;
using Aerospike.Client;
 
namespace Test
{
    public class SumTest
    {
        public static void RunTest()
        {
            AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);
 
            try
            {
                string ns = "test";
                string setName = "demoset";
                string indexName = "aggindex";
                string binName = "aggbin";
 
                string luaDirectory = @"..\..\..\udf";
                LuaConfig.PackagePath = luaDirectory + @"\?.lua";
 
                // Initialize aggregation function and create secondary index.
                string filename = "sum_example.lua";
                string path = Path.Combine(luaDirectory, filename);
                RegisterTask rt = client.Register(null, path, filename, Language.LUA);
                rt.Wait();
 
                IndexTask it = client.CreateIndex(null, ns, setName, indexName, binName, IndexType.NUMERIC);
                it.Wait();
 
                // Write 10 records.
                for (int i = 1; i <= 10; i++)
                {
                    Key key = new Key(ns, setName, "key" + i);
                    Bin bin = new Bin(binName, i);
                    client.Put(null, key, bin);
                }
 
                // Query on secondary index.
                Statement stmt = new Statement();
                stmt.SetNamespace(ns);
                stmt.SetSetName(setName);
                stmt.SetBinNames(binName);
                stmt.SetFilters(Filter.Range(binName, 4, 7));
 
                ResultSet rs = client.QueryAggregate(null, stmt, "sum_example", "sum_single_bin", Value.Get(binName));
 
                try
                {
                    if (rs.Next())
                    {
                       Console.WriteLine("Sum = " + rs.Object);
                    }
                }
                finally
                {
                    rs.Close();
                }
            }
            finally
            {
                client.Close();
            }
        }
    }
}
```

### Understanding the Aggregation

The important components in this example C# program are:

- Defining the Lua files location.
	The user defined `sum_example.lua` must be available for execution on the client. 
-- Set `LuaConfig.PackagePath` to the location of your Lua files.
-- The question mark (`?`) is a wildcard. 
-- Separate multiple Lua search paths using a semicolon (`;`). 

```cs
LuaConfig.PackagePath = @"dir1\?.lua;dir2\?.lua";
string luaDirectory = @"..\..\..\udf";
LuaConfig.PackagePath = luaDirectory + @"\?.lua";
```

- Register `sum_example.lua` with the server. 
-- The second argument is the path to the Lua file on the client.
-- The third argument is the target filename of the Lua file on the server. 

```cs
string filename = "sum_example.lua";
string path = Path.Combine(luaDirectory, filename);
RegisterTask rt = client.Register(null, path, filename, Language.LUA);
rt.Wait();
```

- Create the secondary index:

```cs
IndexTask it = client.createIndex(null, namespace, setName, indexName, binName, IndexType.NUMERIC);
it.Wait();
```

- Define the query and execute aggregation.
-- The client searches for `sum_example.lua` in the specified source directory.
-- The `sum_single_bin` aggregation method is then called with the bin name as an argument.

```cs
// Query on secondary index.
Statement stmt = new Statement();
stmt.SetNamespace(ns);
stmt.SetSetName(setName);
stmt.SetBinNames(binName);
stmt.SetFilters(Filter.Range(binName, 4, 7));

ResultSet rs = client.QueryAggregate(null, stmt, "sum_example", "sum_single_bin", Value.Get(binName));
//To process the result of the aggregation:  (this particular aggegration returns a single object)

try
{
    if (rs.Next())
    {
        Console.WriteLine("Sum = " + rs.Object);
    }
}
finally
{
    rs.Close();
}
```

