---
title: Introduction - Python Client
description: Use the Aerospike Python client to build applications to store and retrieve data with the Aerospike database.
---

Use the Aerospike Python client to build applications to store and retrieve data from an Aerospike cluster.

The Python client is a CPython module, built on the Aerospike C client.

The Python client for Aerospike works with Python 2.7, 3.4, 3.5, 3.6 running on 64-bit OS X 10.9+ and Linux.

## Code

This example imports the Aerospike Python module, creates a client and connects to the cluster, creates a key for record ID, writes and reads a record, and closes the cluster connection.

```python

# import the module
from __future__ import print_function
import aerospike

# Configure the client
config = {
  'hosts': [ ('127.0.0.1', 3000) ]
}

# Create a client and connect it to the cluster
try:
  client = aerospike.client(config).connect()
except:
  import sys
  print("failed to connect to the cluster with", config['hosts'])
  sys.exit(1)

# Records are addressable via a tuple of (namespace, set, key)
key = ('test', 'demo', 'foo')

try:
  # Write a record
  client.put(key, {
    'name': 'John Doe',
    'age': 32
  })
except Exception as e:
  import sys
  print("error: {0}".format(e), file=sys.stderr)

# Read a record
(key, metadata, record) = client.get(key)

# Close the connection to the Aerospike cluster
client.close()
```

## The Data Model

At the top of the data model is the *namespace* container with one set of policy rules for all its data. This is similar to the _database_ concept in an RDBMS, only Aerospike is distributed across the cluster. Namespaces are subdivided into *sets*, which are similar to RDBMS _tables_.

*Bins*  are pairs of key-value data and comprise *records*, which are similar to _columns_ of a _row_ in RDBMS. Aerospike is schema-less. There is no need to define bins in advance.

*Records* are uniquely identified by their *key*, and *sets* have a primary index that contains the for of all records in the set.

<div style="text-align: center;">
<a class="button primary" href="/docs/client/python/start">GET STARTED</a>
</div>
