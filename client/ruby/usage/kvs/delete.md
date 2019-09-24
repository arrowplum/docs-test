---
title: Delete Records
description: Use the Aerospike Ruby client APIs to delete records from the Aerospike database.
---

To delete an entire record from the database using `client.delete`, and delete the key *mykey* in namespace *test* within set *myset*:

```ruby
# Delete a record.
mykey = Key.new('test', 'myset', 'value')

existed = client.delete(mykey)
```

The return value indicates if the key existed in the database.
