---
title: Applying UDFs on Records
description: Use the Aerospike Python client to apply a user-defined function against a record in the Aerospike database.
---

The Aerospike Python client `apply()` API to apply a user-defined function (UDF) against a record in the database.

{{#note}}
UDF modules must register with the database. 
{{/note}}

Provide the following in your application to apply a UDF on a record:

- key tuple &mdash; Address the record in the database.
- module name &mdash; The UDF module.
- function name &mdash; The UDF function.
- function arguments &mdash; Arguments for the UDF function.

### Code

The *sample.lua* UDF module contains the following function to append a value to a bin containing a list and update the record:

```lua
function list_append(rec, bin, value)
  local l = rec[bin]
  list.append(l, value)
  rec[bin] = l
  aerospike:update(rec)
  return 0
end
```

To register the UDF module with the cluster:

```python
client.udf_put('sample.lua')
```

To invoke `list_append()` and append *car* to the bin *things*:

```python
# key tuple
key = ('test', 'demo', 'foo')

# apply the UDF idenfified by `udf_module` and `udf_function`
client.apply(key, "sample", "list_append", ["things", "car"])
```

The UDF return value is in `client.apply()`.

