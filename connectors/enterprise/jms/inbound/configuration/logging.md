---
title: Aerospike Connect Logging Configuration
description: Configuring Aerospike Connect logging section
---

## Logging

This section controls connector log output. Generated log files are rotated daily and by default last 30 days worth of logs are retained.

Option | Required | Default | Description
---| --- | --- | ---
file | no | | The output log file path. Rotated logs will be placed in the same folder as this log file.
max-history | no | 30 | Maximum number of log files to retain.
levels | no | | Map from logger name to its log level. Valid log levels are error, warn, info, debug, trace.
ticker-interval | no | 10 | Ticker log interval in seconds.

### Example

```
logging:
  file: /var/log/aerospike-jms-inbound/aerospike-jms-inbound.log
  levels:
    root: debug # Set default logging level to debug.
    com.solacesystems: error # Set solace package's logging level to error.
```

## Parsing Errors

To log the messages which could not be parsed set the log level of
_message-parser_ to _error_.

### Example

```
logging:
  file: /var/log/aerospike-jms-outbound/aerospike-jms-outbound.log
  levels:
    ...
    message-parser: error # Set message parser logs.
    ...
```