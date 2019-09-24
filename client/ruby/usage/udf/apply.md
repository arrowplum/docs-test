---
title: Apply UDF on a Record
description: UDFs that execute on a single row are Record UDFs, which can update or create a record in the database.
---

UDFs that execute on a single record are Record UDFs. The record may or may not exist in the database, which allows the UDF to update or create a record.

To invoke a Record UDF using `Client.execute_udf`: 

```ruby
Client.execute_udf(
    key, 
    packageName, 
    functionName, 
    args = [],
    opts = {}
)
```

Where:
- `key` &mdash; The key of the record on which to invoke the function.
- `packageName` &mdash; The UDF module that contains the function to invoke.
- `functionName` &mdash; The function to invoke. 
- `args` &mdash; The function arguments.
- `opts` &mdash; The policy options.

This example defines a UDF in the module _examples.lua_:

```lua
function readBin(r, name)
    return r[name]
end
```

`readBin` returns the value of record _r_ in bin _name_.

The client application can invoke `readBin` on a record:

```ruby
result = client.execute_udf(key, "examples", "readBin", ["name"])
```

`key` specifies the record to pass to the UDF as the parameter `r`. 

### Multiple Arguments

If the UDF accepts multiple arguments, add each argument to `Client.execute_udf`.

For example, if the following UDF is defined in _example.lua_:

```lua
function multiplyAndAdd(r, a, b)
    return r['factor'] * a + b;
end
```

(This multiplies the bin _factor_ by _a_ and adds _b_, and returns results to the caller.)

Then, to invoke `multiplyAndAdd()` from Ruby, run:

```ruby
result = client.execute_udf(key, "examples", "multiplyAndAdd", [10, 5])
```

