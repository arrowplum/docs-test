---
title: Apply UDFs on a Record
description: Use the Aerospike Node.js client to execute UDFs on records in the Aerospike database.
---

Use the Aerospike Node.js client `apply()` method to execute UDFs on records in the Aerospike database. 

{{#note}}
UDF modules must register with the Aerospike server.
{{/note}}

Provide the follwing to apply a UDF on a record in the database:

- **key tuple** &mdash; To address the record in the database.
- **module name** &mdash; The UDF module.
- **function name** &mdash; The UDF function.
- **function arguments** &mdash; The UDF arguments.

This example uses the registered UDF module *sample*, which contains the following function:

```lua
function list_append(rec, bin, value)
  local l = rec[bin]
  list.append(l, value)
  rec[bin] = l
  aerospike:update(rec)
  return 0
end
```

`list_append()` appends a value to a bin containing a list, and then updates the record.

To invoke `list_append()` and append *car* to the bin *things*:

```js
# key tuple
var key = new Key('test', 'demo', 'foo')
var udf = { module: 'sample', funcname: 'list_append', args: ['things', 'car']}
// apply the UDF idenfified by `sample` and `list_append`
client.apply(key, udf, function (error, result) {
  if (error) throw error
  // result is the value returned by UDF function. In this case value returned by list_append, i.e. 0
})
```
