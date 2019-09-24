---
title: Logging
description: The Aerospike Ruby client contains a logging interface useful for debugging and informational purposes.
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
  - ruby
---

The Aerospike Ruby client contains a logging interface useful for debugging and informational purposes. By default, logging is disabled. 

### Setting the Logger

To enable the log facility, provide an instance of `Logger`:

```ruby
# Log to a StringIO instance to make sure no exceptions are raised by our
# logging code.
Aerospike.logger = Logger.new(STDOUT, Logger::DEBUG)
```

Standard `Logger` levels control the log verbosity level.

### Rails

If the gem is loaded in Rails, the Rails logger is automatically selected.
