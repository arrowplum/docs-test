---
title: Register a UDF
description: Use the Aerospike UDF API with the Aerospike C# client to register UDFs with the Aerospike server. 
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

To register UDFs, you must first group them in a module and then register that UDF module with the Aerospike server using `AerospikeClient.Register()`:

```cs
RegisterTask task = client.Register(policy, clientPath, serverPath, Language.LUA);
task.Wait();
```

Where,


- `clientPath` &mdash; The local path to the UDF module.
- `serverPath` &mdash; The UDF module name to store in the cluster.
- `language` &mdash; The UDF module language.

UDF modules register asynchronously, which means that the server can return before the UDF module is available on all nodes. The client can use `Wait()` to wait until the asynchronous task completes:

Once registration is complete, the UDF functions can be called from any Aerospike client.

{{#note}}
Registering the Lua package only needs to happen once.
{{/note}}

