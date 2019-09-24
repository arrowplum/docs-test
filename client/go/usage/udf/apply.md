---
title: Apply UDF on a Record
description: Use the Aerospike Go client UDFs to extend the functionality and performance of the Aerospike database. 
---

User-defined functions (UDFs) that execute on a single row are Record UDFs. Record UDFs *atomically* update or create a record in the database. 

To invoke a UDF against a record in the database using the `Client.Execute()` method: 

```go
func (clnt *Client) Execute(policy *WritePolicy, 
    key *Key, 
    packageName string, 
    functionName string, 
    args ...Value) (interface{}, error)
```

Where,

- `key` &mdash; The key of the record on which to invoke the function.
- `packageName` &mdash; The name of the UDF module that contains the function to invoke.
- `functionName` &mdash; The name of the function to invoke. 
- `args` &mdash; The arguments for the function to invoke.

This example has a UDF defined in the module _examples.lua_:

```lua

function readBin(r, name)
    return r[name]
end
```

`readBin` returns the value of bin _name_ in record _r_.

To invoke `readBin` from your application on a record:

```go

result, err := client.Execute(
    nil, key, "examples", "readBin", NewValue("name"))
```

`key` is the record ID to pass as parameter _r_ to the UDF. 

### Multiple Arguments

If the UDF accepts multiple arguments, add each argument to the `client.Execute()` method call.

This example has a UDF defined in the module _examples.lua_:

```lua
function multiplyAndAdd(r, a, b)
    return r['factor'] * a + b;
end
```

This multiplies the bin _factor_ by _a_ and adds _b_, and then returns the result to the caller.

To invoke `multiplyAndAdd()` function from Go:

```go

result, err := client.Execute(
    nil, key, "examples", "multiplyAndAdd", NewValue(10), NewValue(5))
```

