---
title: Error Handling
description: Use the Aerospike PHP client to manage errors.
---

Use the Aerospike PHP client to manage errors.

The Aerospike PHP client passes the [status code](https://github.com/aerospike/aerospike-client-c/blob/master/src/include/aerospike/as_status.h) that returns from the underlying C client. All non-zero status codes (`!== Aerospike::OK`) are errors.

Status codes are constants of the [Aerospike class](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike.md).

```php
$status = $db->get($key, $record);
if ($status == Aerospike::OK) {
    var_dump($record);
} elseif ($status == Aerospike::ERR_RECORD_NOT_FOUND) {
    echo "A record with this key does not exist in the database\n";
}
```

### Determining Operation Status

Use these calls to determine the operation status of the most-recent client operation:

- [error()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_error.md)
- [errorno()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_errorno.md)

```php
$db = new Aerospike($config);
if (!$db->isConnected()) {
   echo "Failed to connect[{$db->errorno()}]: {$db->error()}\n";
   exit(1);
}
```

