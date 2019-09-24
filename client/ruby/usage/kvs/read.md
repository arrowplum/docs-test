---
title: Read Records
description: Use Aerospike Ruby client APIs to read records from the Aerospike database.
---

Use Aerospike Ruby client `#get` APIs to:

- Read all record bins.
- Read only specified record bins.
- Read only the record metadata.
- Check existence of a record.

Each method returns a `Record` object containing the specified record metadata and bins.

See [Batch Reads](/docs/client/ruby/usage/kvs/batch.html) to learn how to read multiple records in the same call.

### Reading All Record Bins

Use `#get` to read all record bins. This reads the record and metadata from the database and returns a `Record` object.

To read all record bins:

```ruby
record = client.get(key)
```

### Reading Specific Record Bins

Use `#get` and specify each bin to read specific bins of a record. This reads the record metadata and returns a `Record` object containing the specified bins.

This example reads the  *name* and *age* bins from the record:

```ruby
record = client.get(key, ['name', 'age'])
```

### Checking Record Existence

To check that a record exists using `#exists`:

```ruby
exists = client.exists(key)
```

A batch request for existence is also available. 

The result array returns in the same order as the key array:

```ruby
keys = []
for i in 0...size do
	keys << Key.new('test', 'myset', (i + 1))
end

existsArray = client.batch_exists(keys)
```
