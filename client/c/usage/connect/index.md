---
title: Connecting
description: Configure and connect the Aerospike C client to the Aerospike cluster.
---

The `aerospike` object represents a single cluster. To connect to a cluster, you must configure an `aerospike` object and initialize the client.

# Configuring the Client

To configure the client, initialize the `as_config` object to its default value using `as_config_init()`:

```cpp
as_config config;
as_config_init(&config);
```

Minimum configuration host requirements are one host to seed the client. The client attempts connect to each seed host until it succeeds. On success, the host connects to one node in the cluster and automatically discovers the remaining cluster nodes.

To populate the `as_config` object with application-specific settings:

```cpp
config.hosts[0] = { .addr = "127.0.0.1", .port = 3000 };
```

# Initializing a Client

To connect to the cluster, initialize an `aerospike` client object using your configuration:

```cpp
aerospike as;
aerospike_init(&as, &config);
```

On success, `aerospike_init()` returns the initialized `aerospike` object; otherwise NULL returns.

# Establishing a Connection

On successful client initialization, you can connect to the cluster. 

On error, `aerospike_connect()` requires an `as_error` object to populate:

```cpp
as_error err;
if (aerospike_connect(&as, &err) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

The value of `err.code` must match the return code from `aerospike_connect`. If `AEROSPIKE_OK` is not the return value, an error occurred. Check the `err` object for information.

An `aerospike` object internally contains the cluster status and maintains the connection pools to the cluster. The application can reuse the `aerospike` object for database operations to a given cluster.

If the application needs to connect to multiple Aerospike clusters, it must create multiple `aerospike` objects, one for each cluster.

# Closing a Connection

When the application no longer requires the client connections to the cluster, use `aerospike_close()` to close connections:

```cpp
as_error err;
if (aerospike_close(&as, &err) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

# Cleaning Up Resources

You can use `aerospike_destroy()` to destroy the client and release all of its resources:

```cpp
aerospike_destroy(&as);
```

