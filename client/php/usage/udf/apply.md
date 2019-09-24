---
title: Apply UDF on Record
description: Use the Aerospike PHP client to apply a UDF on a record.
---

Use the Aerospike PHP client to apply a UDF on a record. Invoke the [apply()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_apply.md) **Record UDF** to perform an operation on a single record. 


{{#note}}
UDFs must register with the Aerospike server.
{{/note}}

Provide the following to apply a UDF on a record:

- **key tuple** &mdash; Address a record in the database.
- **module name** &mdash; The UDF module.
- **function name** &mdash; The UDF function.
- **function arguments** &mdash; The arguments for the UDF function.

The example uses the registered a UDF module, `sample`, which contains the following function:

```lua
function list_append(rec, bin, value)
  local l = rec[bin]
  list.append(l, value)
  rec[bin] = l
  aerospike:update(rec)
  return 0
end
```

`list_append()` appends a value to the bin containing a list, and then updates the record. 

To invoke `list_append()` and append "!!!" to the *things* bin:

```php
$key = $db->initKey('test', 'demo', 'foo');
$db->apply($key, "sample", "list_append", ["things", "!!!"]);
```

See the [User-Defined Functions (UDF) Development Guide](/docs/udf/udf_guide.html).
