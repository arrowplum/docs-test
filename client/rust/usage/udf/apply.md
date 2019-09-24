---
title: Apply UDF on a Record
description: UDFs that execute on a single row are Record UDFs, which can update or create a record in the database.
---

UDFs that execute on a single record are Record UDFs. The record may or may not
exist in the database, which allows the UDF to update or create a record.

To invoke a Record UDF, use `Client.execute_udf()`: 

```rust
pub fn execute_udf(&self,
                           policy: &WritePolicy,
                           key: &Key,
                           udf_name: &str,
                           function_name: &str,
                           args: Option<&[Value]>)
                           -> Result<Option<Value>>;
```

Where,

- `policy` &mdash; The policy governing the operation.
- `key` &mdash; The key of the record on which to invoke the function.
- `udf_name` &mdash; The UDF module that contains the function to invoke.
- `function_name` &mdash; The function to invoke. 
- `args` &mdash; The optional function arguments.

This example defines a UDF in the module _examples.lua_:

```lua
function readBin(r, name)
    return r[name]
end
```

`readBin` returns the value of record _r_ in bin _name_.

The client application can invoke `readBin` on a record:

```rust
let result = client.execute_udf(&WritePolicy::default(),
    &key, "examples", "readBin", as_values!("name"));
```

`key` specifies the record to pass to the UDF as the parameter `r`. 

### Multiple Arguments

If the UDF accepts multiple arguments, add each argument to `client.execute_udf()`. 

For example, if the following UDF is defined in _example.lua_:

```lua
function multiplyAndAdd(r, a, b)
    return r['factor'] * a + b;
end
```

(This multiplies the bin _factor_ by _a_ and adds _b_, and returns results to the caller.)

Then, to invoke `multiplyAndAdd()` from Rust, run:

```rust
let result = client.execute_udf(&WritePolicy::default(),
    &key, "examples", "multiplyAndAdd", as_values!(10, 5));
```
