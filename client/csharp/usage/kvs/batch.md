---
title: Batch Reads
description: Run batch record reads using the Aerospike C# client on the Aerospike database.
---

Multiple records can be read in a single batch call.

```cs
Key[] keys = new Key[size];

for (int i = 0; i < 1000; i++) {
    keys[i] = new Key("test", "myset", (i + 1));
}

Record[] records = client.Get(policy, keys);
```

This call groups the keys based on which Aerospike server node can best handle the request, and uses the C# thread pool to concurrently handle the requests to all nodes. After all the nodes return the record data, the records return to the caller.  The array of records returned is in the same order as the key passed in.  If a record is not found on the database, the array entry is null.
