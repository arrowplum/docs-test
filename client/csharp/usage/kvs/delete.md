---
title: Delete a Record
description: Use the Aerospike C# client to delete records from the Aerospike database.
---

This example deletes the entire record _test_ within the set _myset_ using _mykey_:

```cs
Key key = new Key("test", "myset", "mykey");

client.Delete(policy, key);
```

