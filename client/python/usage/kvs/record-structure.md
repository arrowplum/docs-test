---
title: Record Structure
description: The Aerospike Python client record structure.
---

<a href="/docs/architecture/data-model.html">Data in Aerospike</a> is organized into *namespaces* (similar to  _databases_ in RDBMS) that contain *sets* (similar to RDBMS  _tables_).  Inside sets are records uniquely identified by a *hash key digest* (20-byte RIPEMD-160). An in-memory primary index over these digest values provides fast access to records.

### Records

Records comprise `bins`, which are named, strongly-typed containers that hold both atomic (string, integer, bytes) and complex (map, list) data types.

Record contain two metadata values:
- `generation` &mdash; the number of times it has been modified) 
- `ttl` &mdash; The seconds remaining until record expiration (default = 0; never expire). 
  - Expired records are garbage-collected by the server. 
  - On a record write or `touch()`, `ttl` resets.

### Structure

This example shows the record structure:

```python
(key, metadata, bins)= client.get((namespace, set, key), policy)
print (key)
print (metadata)
print (bins)
```

Where,
- `key: \(_namespace_, _set_, _key_, _digest_\)`
- `metadata: \{'gen': _generation_, 'ttl': _ttl_\}`
- `bins` &mdash; Is the  _key-value pairs of data_ bins dictionary

The following structure returns:

```bash
('test', 'demo', u'1', bytearray(b"\xa9\x84\'\xc16y\x99T\xd6d\xcc\xc6\x07\xc4\xa5 n&s\xdf"))
{'gen': 1, 'ttl': 4294967295}
{'a': 1, 'foo': u'bar', 'z': u'26'}
```

- `_digest_` is in the key tuple even if `_key_` is `None`. 
- `_digest_` is the actual record unique identifier. 
- The hashed string or integer input to produce `_digest_` may or may not be stored with the record, as set in the key policy:

```python
policy = {'key': aerospike.POLICY_KEY_SEND} # store the key along with the record
meta = {'ttl': 120}
client.put((namespace, set, key), bins, meta, policy)
```

