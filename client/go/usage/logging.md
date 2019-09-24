---
title: Logging
description: Use the Aerospike Go client logging interface for debugging and informational purposes. 
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
  - go
---

Use the Aerospike Go client logging interface for debugging and informational purposes. The log is disabled by default. 

### Setting the Logger

To enable the log facility, provide an instance of `*log.Logger`:

```go
import (
  "log"

  asl "github.com/aerospike/aerospike-client-go/logger"
)

var buf bytes.Buffer
logger := log.New(&buf, "logger: ", log.Lshortfile)
asl.SetLogger(logger)

```

### Setting the Log Level

To control the verbosity of logs sent to the logger, use `logger.SetLevel()`:

```go
import asl "github.com/aerospike/aerospike-client-go/logger"

asl.Logger.SetLevel(asl.INFO)
```

Log levels are defined in _[logger/logger.go](https://github.com/aerospike/aerospike-client-go/blob/master/logger/logger.go)_.

**Example**

To log all debug messages, set the level to `asl.DEBUG`:

```go
import asl "github.com/aerospike/aerospike-client-go/logger"

asl.Logger.SetLevel(asl.DEBUG)
```

