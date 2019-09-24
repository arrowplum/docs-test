---
title: Manage UDFs
description: Use the Aerospike Nodejs client to asynchronously register or delete UDF modules with the Aerospike server.
---

Use the Aerospike Nodejs client to asynchronously register or delete UDF modules with the Aerospike server.

User-defined functions (UDFs) are grouped into modules. UDF modules must register with the Aerospike server. 

### Registering a UDF Module

To register the *example.lua* UDF module using `client.udfRegister()`:

```js
client.udfRegister('path/to/example.lua', function (error) {
  if (error) {
    console.error('Error: %s [%d]', error.message, error.code)
  }
})
```

Once registration completes, the functions in the *example.lua* UDF module are available to any Aerospike client.

### Deleting a UDF Module

When a UDF module is no longer required, delete it from the Aerospike server.

To remove the *example.lua* UDF module using `client.udfRemove()`:

```js
client.udfRemove('example.lua', function (error) {
  if (error) {
    console.error('Error: %s [%d]', error.message, error.code)
  }
})
```
