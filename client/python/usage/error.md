---
title: Error Handling
description: Use the Aerospike Python client to detect errors when using the Aerospike database.
---

On error, the Aerospike Python client raises an [`AerospikeError`](/apidocs/python/exception.html) exception or one of its subclasses. The exception includes the following attributes:

- `code` &mdash; The error code that identifies the error. See [as_status.h of the Aerospike C client](/apidocs/c/dc/d42/as__status_8h_source.html) for error code descriptions.
- `msg` &mdash; The error message with details.
- `file` &mdash; The file where the error occurred.
- `line` &mdash; The line number in the file where the error occurred.

### Exception Handling

To handle the exceptions, wrap the code in a `try-except` block:

```python
from __future__ import print_function

import aerospike
from aerospike.exception import *

try:
  config = { 'hosts': [ ('127.0.0.1', 3000)], 'policies': { 'timeout': 1200}}
  client = aerospike.client(config).connect()
  client.close()
except ClientError as e:
  print("Error: {0} [{1}]".format(e.msg, e.code))
```

