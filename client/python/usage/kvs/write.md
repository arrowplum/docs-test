---
title: Writing Records
description: Use the Aerospike Python client APIs to write data to the Aerospike database.
---

Use the Aerospike Python client `put()` API to store data in the Aerospike cluster.

### Defining the Key

Define the key tuple as the record unique identifier.

This example defines a key tuple in the set *characters* with the primary key *bender* in namespace *test*:

```python
# create the key tuple identifying the record
key = ('test', 'characters', 'bender')
```

### Specifying Record Data

Specify record data in a `dict`, where the top-level object fields represent the bin names, and field values correspond to bin values.

This example writes six bins: *name*, *serialnum*, *lastsentence*, *composition*, *apartment*, and *quote_cnt*:

```python
# The record data to write to the cluster
bins = {
  'name': 'Bender',
  'serialnum': 2716057,
  'lastsentence': {
    'BBS': "Well, we're boned",
    'TBwaBB': 'I love you, meatbags!',
    'BG': 'Whip harder, Professor!',
    'ltWGY': 'Into the breach, meatbags. Or not, whatever'},
  'composition': [ "40% zinc", "40% titanium", "30% iron", "40% dolomite" ],
  'apartment': bytearray(b'\x24'),
  'quote_cnt': 47
}
```

### Storing the Record

To store a record in the database using `put()`:

```python
# Put the record to the database.
client.put(key, bins)
```

#### Appending, Prepending and Incrementing a Bin

Use the following APIs to implement prepending, appending, and incrementing bins:

```python
client.prepend(key, 'name', 'Dr. ')
client.append(key, 'name', ' Bending Rodriguez')
client.increment(key, 'quote_cnt', 3)
```

#### Multi-Ops

To combine multiple bin operations on a single record with post-write reads for the modified bin values:

```python
operations = [
  {
    "op" : aerospike.OPERATOR_INCR,
    "bin" : "quote_cnt",
    "val" : -1
  },
  {
    "op" : aerospike.OPERATOR_READ,
    "bin" : "quote_cnt"
  }
]
(key, meta, bins) = client.operate(key, operations)
```

#### References

Read the following APIs for more information:

 - <a href="/apidocs/python/client.html#aerospike.Client.put" target="_api">aerospike.Client.put()</a>
 - <a href="/apidocs/python/client.html#aerospike.Client.append" target="_api">aerospike.Client.append()</a>
 - <a href="/apidocs/python/client.html#aerospike.Client.prepend" target="_api">aerospike.Client.prepend()</a>
 - <a href="/apidocs/python/client.html#aerospike.Client.increment" target="_api">aerospike.Client.increment()</a>
 - <a href="/apidocs/python/client.html#aerospike.Client.operate" target="_api">aerospike.Client.operate()</a>

#### Example

This example combines write oprations:

```python
# -*- coding: utf-8 -*-
from __future__ import print_function
import aerospike
import pprint
import sys

try:
    config = {
        'hosts': [ ('127.0.0.1', 3000) ]
    }
    client = aerospike.client(config).connect()
except Exception as e:
    print("error: {0}".format(e), file=sys.stderr)
    sys.exit(1)

try:
    pp = pprint.PrettyPrinter(indent=2)
    client = aerospike.client(config).connect()
    key = ('test', 'characters', 'bender')
    bins = {
        'name': 'Bender',
        'serialnum': 2716057,
        'lastsentence': {
            'BBS': "Well, we're boned",
            'TBwaBB': 'I love you, meatbags!',
            'BG': 'Whip harder, Professor!',
            'ltWGY': 'Into the breach, meatbags. Or not, whatever'},
        'composition': [ "40% zinc", "40% titanium", "30% iron", "40% dolomite" ],
        'apartment': bytearray(b'\x24'),
        'quote_cnt': 47
    }
    client.put(key, bins,
               meta={'ttl':60})

    (key, meta, bins) = client.get(key)
    pp.pprint(key)
    pp.pprint(meta)
    pp.pprint(bins)

    print('-----------------------------------------------------------')
    client.prepend(key, 'name', 'Dr. ')
    client.append(key, 'name', ' Bending Rodriguez')
    client.increment(key, 'quote_cnt', 3, meta={'ttl':meta['ttl']})
    (key, meta, bins) = client.get(key)
    pp.pprint(meta)
    pp.pprint(bins)

    print('-----------------------------------------------------------')
    operations = [
        {
            'op' : aerospike.OPERATOR_INCR,
            'bin' : 'quote_cnt',
            'val' : -1
        },
        {
            'op' : aerospike.OPERATOR_READ,
            'bin' : 'quote_cnt'
        }
    ]
    (key, meta, bins) = client.operate(key, operations, meta={'ttl': meta['ttl']})
    pp.pprint(meta)
    pp.pprint(bins)
    client.close()

except Exception as e:
    print("error: {0}".format(e), file=sys.stderr)
    client.close()
    sys.exit(2)

```
