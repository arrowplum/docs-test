---
title: Delete a Record
description: Use the Aerospike Java client to delete records from the Aerospike database.
---

This example deletes the key _mykey_ from the namespace _test_ in the set _myset_:

```java
Key key = new Key("test", "myset", "mykey");
client.delete(policy, key);
```
