---
title: Logging
description: Use the Aerospike Node.js client to control log levels for your application. 
---

The Aerospike Node.js client has internal logging. Use the `Log` object to control the application log verbosity level and specify the location for storing log messages. By default, `Log` is set to `INFO` to send its log messages to stderr.

Log verbosity levels are defined in *aerospike.log*:

- `OFF`
- `ERROR`
- `INFO` (default)
- `DEBUG`
- `DETAIL`

### Configure Logging

Configure client logging when setting system configuration. In the client configuration object, specify a `log` field to contain the following objects:

- `level` — The log verbosity level.
- `file` — The path to the log file.

This is an example configuration with defined log settings:

```js
const Aerospike = require('aerospike')

var config = {
  log: {
    level: aerospike.log.INFO,
    file: "/var/log/myapplication.log"
  }
}

Aerospike.connect(config, function (error, client) {
  // ...
})
```

### Update Logging

Modify client logging during run time by using `client.updateLogging()`, which takes an object.

To reset the log level while the application is running:

```js
client.updateLogging({level: aerospike.log.DEBUG})
```

