---
title: Batch Reads
description: Use the Aerospike Java client to read multiple records in one sweep from the Aerospike Database.
---

Multiple records can be read in a single batch call.

```java
Key[] keys = new Key[size];

for (int i = 0; i < size; i++) {
	keys[i] = new Key("test", "myset", (i + 1));
}

Record[] records = client.get(policy, keys);
```

This call groups keys based on which Aerospike Server node can best handle the request, and uses a `ThreadPool` object to concurrently handle each request to each node. After all nodes return the record data, the records are returned to the caller. The array of records returned is in the same order the keys are passed in. If a record is not found in the database, the array entry is null.
