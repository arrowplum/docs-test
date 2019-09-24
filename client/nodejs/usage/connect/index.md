---
title: Connecting
description: The Aerospike Nodejs client can connect and periodically ping nodes for cluster status. 
---

The Aerospike Nodejs client can connect and periodically ping nodes for cluster status.

To connect to the Aerospike cluster, you must:
- Import the module into your application.
- Configure a client instance.
- Connect to the cluster.

### Import the Module

To enable the Aerospike Node.js client, import the module into your application:

```js
const Aerospike = require('aerospike')
```

### Creating a Client

Before making any database operation, you must configure a client object, and successfully connect to the database.

To create a client, initialize the client using a `configuration` object and specify the following options:
- `username` &mdash; The username to login to the Aerospike cluster (only available in security-feature clusters).
- `password` &mdash; The password for the username (only available in security-feature clusters).
- `hosts` &mdash; Specify an array of hosts to connect to. The client iterates over the list until it successfully connects with a server.
- `log` &mdash; Specify the log verbosity level and redirect log messages to a file.
- `policies` &mdash; Specify client default behavior.

This example connects to an Aerospike cluster:

```js
let client = Aerospike.client({
    hosts: [
        { addr: "127.0.0.1", port: 3000 }
    ],
    log: {
        level: aerospike.log.INFO
    }
})
```

On successful client creation, you can connect and execute operations in the cluster.

### Connecting to the Cluster

To connect to the Aerospike cluster:

```js
client.connect(function (error) {
  if (error) {
    // handle failure
    console.log('Connection to Aerospike cluster failed!')
  } else {
    // handle success
    console.log('Connection to Aerospike cluster succeeded!')
  }
})
```

Your application is ready to execute database operations.
