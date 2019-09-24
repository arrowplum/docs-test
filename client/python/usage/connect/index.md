---
title: Connecting
description: Use the Aerospike Python client APIs to connect to the Aerospike database.
---

Use the Aerospike Python client APIs to connect to the Aerospike database.

### Import the Module

To import the Aerospike Python client module in to your application:

```python
import aerospike
```

### Configuring a Client

To create a configuration for the client that specifies various options, including:

- `hosts` &mdash; An array of (address, port) tuples that describe the cluster (only one tuple is required).
- `policies` &mdash; A dictionary of the following policies:
  - `timeout` &mdash; The default timeout in milliseconds.
  - `key` &mdash; The default key policy for this client.
  - `exists` &mdash; The default exists policy for this client.
  - `gen` &mdash; The default generation policy for this client.
  - `retry` &mdash; The default retry policy for this client.
  - `consistency_level` &mdash; The default consistency level policy for this client.
  - `replica` &mdash; The default replica policy for this client.
  - `commit_level` &mdash; The default commit level policy for this client.

This example connects to an Aerospike cluster:

```python
config = {
    'hosts': [
        ( '127.0.0.1', 3000 )
    ],
    'policies': {
        'timeout': 1000 # milliseconds
    }
}
```

### Creating a Client

To create a new client:

```python
client = aerospike.client(config)
```

Once the client is created, you can connect and execute operations against the cluster.

### Connecting to the Cluster

To connect to the cluster:

```python
client.connect()
```

A failed connection results in an exception.
On success, your application can execute database operations.

#### References

Read the following APIs for more information:
 
- <a href="/apidocs/python/aerospike.html#aerospike.client" target="_api">aerospike.client()</a>
- <a href="/apidocs/python/client.html#aerospike.Client" target="_api">aerospike.Client.connect()</a>


