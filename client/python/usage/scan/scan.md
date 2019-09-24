---
title: Scan
description: Use the Aerospike Python client APIs to scan records in the Aerospike database.
---

Use the Aerospike Python client APIs to scan records in the Aerospike database using the primary index.

### Creating a Scan

Create a new scan using `client.scan()`. `client.scan()` takes the `namespace` (required) and `set` (optional) arguments. `set` can be ommited or `None`. The return value is a new `aerospike.Scan` class instance.

This example creates a query on namespace *test* and set *demo*:

```python
scan = client.scan('test', 'demo')
```

### Projecting Bins

Project bins using `select()` on the `Scan` class instance. `select()` accepts one or many bin names (strings).

This example selects the *name* and *age* bins from the specified records:

```python
scan.select('name', 'age')
```

### Reading Results

Execute the scan and read the results using `foreach()` in the `Scan` class instance. `foreach()` accepts a callback function called for each result read in the scan. The callback function must accept a single argument as a tuple:

- `key tuple` &mdash; The record ID tuple.
- `metadata` &mdash; The dict containing the record metadata (TTL and generation) values.
- `record`  &mdash; The dict containing the reocrd bins.

If the callback returns `False`, then the client stops reading results.

This example executes a query and reads results:

```python
# callback function will print the records as they are read
def print_result((key, metadata, record)):
    print(key, metadata, record)

# Execute the scan and call print_result for each result
scan.foreach(print_result)
```

