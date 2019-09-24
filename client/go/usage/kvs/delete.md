---
title: Delete a Record
description: Use the Aerospike Go client to delete records from the Aerospike database.
---

Use the Aerospike Go client to delete records from the Aerospike database.

To delete an entire record from the database:

```go
// Delete record.
mykey, _ := as.NewKey("test", "myset", "mykey")

existed, err := client.Delete(wpolicy, mykey)
```

This deletes the key _mykey_ in namespace _test_ within set _myset_.
