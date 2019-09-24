---
title: Manage UDFs
description: Use the Aerospike PHP client to manage UDFs on the Aerospike server.
---

Use the following Aerospike PHP client APIs to manage user-defined functions (UDFs) on the server:

- [register()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_register.md)
- [deregister()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_deregister.md)
- [listRegistered()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_listregistered.md)
- [getRegistered()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_getregistered.md)

```php
$status = $db->register('/path/to/my_udf.lua', 'my_udf');
if ($status == Aerospike::OK) {
    echo "UDF module at $path is registered as my_udf on the Aerospike DB.\n";
    $status = $db->listRegistered($modules);
    if ($status == Aerospike::OK) {
        var_dump($modules);
    }
} else {
    echo "Error[{$db->errorno()}]: ".$db->error();
}
```

