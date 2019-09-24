---
title: Connecting
description: The Aerospike PHP client connection methods.
---

This describes the preferred Aerospike PHP client connection methods.

### Configuring the Client

The Aerospike PHP client is cluster-aware. The client learns cluster topology from any node connection and tracks cluster status.

The Aerospike PHP client automatically shards data. See [Data Distribution](/docs/architecture/data-distribution.html).

The _Aerospike_ class constructor configuration must follow the format:

- `hosts` &mdash; A host data array. 
- `addr` &mdash; Node hostname or IP.
- `port`

```php
$config = [
    "hosts" => [
        ["addr" => "127.0.0.1", "port" => 3000]
    ]
];
```

### Creating a Client

The [constructor](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_construct.md) uses the configuration to initialize the cluster connection.

```php
$db = new Aerospike($config);
```

### Verifying the Connection

To verify the connection using [isConnected()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_isconnected.md):

```php
if (!$db->isConnected()) {
  echo "Failed to connect to the Aerospike server [{$db->errorno()}]: {$db->error()}\n";
  exit(1);
}
```

On successful connection, the client is ready for your application to perform database operations.


