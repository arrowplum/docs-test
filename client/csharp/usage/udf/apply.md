---
title: Apply UDF on a Record
description: A user-defined function (UDF) executes on a single row (a Record UDF) to update or create a record.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

A user-defined function (UDF) executes on a single row (a **Record UDF**) to update or create a record. The record may or may not exist in the database, allowing the UDF to update or create a record.

To invoke a UDF against a record in the database using `AerospikeClient.Execute()`: 

```cs
string received = (string)client.Execute(policy, key, packageName, functionName, args);
```

Where,

- `key` &mdash; The key of the record on which the function will invoked.
- `packageName` &mdash; The name of the UDF module containing the function to invoke.
- `functionName` &mdash; The name of the function to invoke. 
- `args` &mdash; The arguments for the function to invoke.

This example is a UDF defined in the _examples.lua_ module:

```lua
function readBin(r, name)
    return r[name]
end
```

`readBin` returns the value the _name_ bin in record _r_.

To invoke `readBin` on a record:

```cs
string result = (string) client.Execute(
    null, key, "examples", "readBin", Value.Get("name")
);
```
Where,

- `key` identifies the record to pass as the parameter `r` to the UDF. 

### Using Multiple Arguments

If the UDF accepts multiple arguments, add each argument to `client.Execute()`. For example, with the following UDF defined in _example.lua_:

```lua
function multiplyAndAdd(r, a, b)
    return r['factor'] * a + b;
end
```

Bin `factor` is multiplied by `a` and adds `b`, and then returns the result to the caller.

To invoke `multiplyAndAdd()` from C#:

```cs
string result = (string) client.Execute(
    null, key, "examples", "multiplyAndAdd", Value.Get(10), Value.Get(5)
);
```

