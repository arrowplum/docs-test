---
title: Reading Records
description: Use the Aerospike Python client APIs to read records in Aerospike database.
---

Use the Aerospike Python client APIs to read records in Aerospike database.

### Defining the Key

Define the key as the record unique identifier.

This example defines a key tuple in the set *demo* with the primary key *foo* in namespace *test*:

```python
# Key of the record
key = ('test', 'demo', 'foo')
```

### Reading a Record

To read a record from the database:

```python
# Retrieve the record using the key.
(key, meta, bins) = client.get(('test','demo','foo'))
```

Where,
- `key` &mdash; The key tuple of the record that was read. 
- `meta` &mdash; The dict containing the record metadata `gen` and `ttl` fields. 
- `bins` &mdash; The dict containing the bins of the record.

`meta` and `bins` are `None` if the record is not found.

### Projecting the Bins of a Record

This example reads specific bins in a record. Note that the second argument is a tuple of bin names to project.

```python
# Retrieve the record using the key.
(key, meta, bins) = client.select(('test','demo','key1'), ('bin1', 'bin2'))
```

Where,
- `key` &mdash; The key tuple of the record that was read. 
- `meta` &mdash; The dict containing the record metadata `gen` and `ttl` fields.
- `bins` &mdash; The dict containing the bins of the record.

`meta` and `bins` are `None` if the record is not found.

### Checking the Existence of a Record

This example checks the existence of a record in the database:

```python
# Retrieve the record using the key.
(key, meta) = client.exists(('test','demo','key1'))
```

`meta` is `None` if the record is not found.

{{#note}}
The `exists` operation is faster than `get()` because it only looks at the index of the record (see <a href="/docs/architecture/primary-index.html">Primary Index</a>).
{{/note}}

### Running Batch Operations

The `get_many` and `exists_many` batch operations allow the application to access multiple keys.

```php
import pprint
pp = pprint.PrettyPrinter(depth=4)
keys = []
for i in xrange(1,5):
  key = ('test', 'demo', 'key' + str(i))
  keys.append(key)
records = client.get_many(keys)
pp.pprint (records)
```

If only two records are found in the given range, the following returns:

```bash
{1: (('test',
      'demo',
      u'key1',
      bytearray(b'\xf5\xce\n8$\xf8<W#\xa0\x9ey\xb2\x16%j\x91\x92\xd5\x85')),
     {'gen': 1, 'ttl': 2588891},
     {'a': 1, 'b': 2}),
 2: (('test',
      'demo',
      u'key2',
      bytearray(b'\xd7\xbb<\xfez\x9d\x9bU~\xad<xr\xa7C\xe8h\x8b\xaa\x9b')),
     {'gen': 1, 'ttl': 2591292},
     {'y': 25, 'z': 26})}

```

#### References

Read the following APIs for more information:

- <a href="https://aerospike-python-client.readthedocs.io/en/latest/client.html#aerospike.Client.exists" target="_api">aerospike.Client.exists()</a>
- <a href="https://aerospike-python-client.readthedocs.io/en/latest/client.html#aerospike.Client.put" target="_api">aerospike.Client.get()</a>
- <a href="https://aerospike-python-client.readthedocs.io/en/latest/client.html#aerospike.Client.select" target="_api">aerospike.Client.select()</a>
- <a href="https://aerospike-python-client.readthedocs.io/en/latest/client.html#aerospike.Client.exists_many" target="_api">aerospike.Client.exists_many()</a>
- <a href="https://aerospike-python-client.readthedocs.io/en/latest/client.html#aerospike.Client.get_many" target="_api">aerospike.Client.get_many()</a>

