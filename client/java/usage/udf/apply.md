---
title: Apply UDF on a Record
description: UDFs that execute on a single row are Record UDFs, which can update or create a record in the database.
---

UDFs that execute on a single record are Record UDFs. The record may or may not exist in the database, which allows the UDF to update or create a record.

To invoke a Record UDF, use `AerospikeClient.execute()`: 

```java
public class AerospikeClient {
    public final Object execute(
        Policy policy,
        Key key,
        String packageName,
        String functionName,
        Value... args
    )   throws AerospikeException
}
```

Where,

- `key` &mdash; The key of the record on which to invoke the function.
- `packageName` &mdash; The UDF module that contains the function to invoke.
- `functionName` &mdash; The function to invoke. 
- `args` &mdash; The function arguments.

This example defines a UDF in the module _examples.lua_:

```lua
function readBin(r, name)
    return r[name]
end
```

`readBin` returns the value of record _r_ in bin _name_.

The client application can invoke `readBin` on a record:

```java
String result = (String) client.execute(
    null, key, "examples", "readBin", Value.get("name")
);
```

`key` specifies the record to pass to the UDF as the parameter `r`. 

### Multiple Arguments

If the UDF accepts multiple arguments, add each argument to `client.execute()`. 

For example, if the following UDF is defined in _example.lua_:

```lua
function multiplyAndAdd(r, a, b)
    return r['factor'] * a + b;
end
```

(This multiplies the bin _factor_ by _a_ and adds _b_, and returns results to the caller.)

Then, to invoke `multiplyAndAdd()` from Java, run:

```java
String result = (String) client.execute(
    null, key, "examples", "multiplyAndAdd", Value.get(10), Value.get(5)
);
```

