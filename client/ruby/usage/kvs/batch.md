---
title: Batch Reads
description: Use the Aerospike Ruby client APIs to read multiple records in one request.
---

Use the Aerospike Ruby client `client.batch_get` API to read multiple records in one request.

### Initializing the Key Array

To set a key array for the application to read:

```ruby
keys = []

for i in 0...10 do
	keys << Key.new('test', 'myset', (i + 1))
end
```
{{#note}}
The keys must all belong to the same namespace.
{{/note}}

### Making the Batch Read Request

To make a batch read request:

```ruby
records = client.batch_get(keys)
```

This call groups keys based on which Aerospike server node can best handle the request, and uses multiple threads to  concurrently handle requests to all nodes. After all the nodes return record data, records return to the caller.

The array of records return in the same order the that key was passed.

If a record is not found in the database, the corresponding array entry is `nil`.

