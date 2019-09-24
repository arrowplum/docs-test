---
title: Deleting Records
description: Use the Aerospike Python client to delete records from the Aerospike database. 
---

Use the Aerospike Python client `remove()` API to delete records from the database. 

This example defines a key object and deletes the corresponding record in the database:

```python
# Key of the record to be deleted
key = ('test', 'demo', 'foo')
 
# Delete the record
client.remove(key)
```

An exception is thrown if the record does not exist.
