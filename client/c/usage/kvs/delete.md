---
title: Delete Records
description: Use the Aerospike C client API to delete data from the Aerospike database.
---

Use `aerospike_key_remove()` to delete data from the database.

These examples are in _examples/basic\_examples/put_ in the Aerospike C client package.

The client must have an [established the cluster connection](/docs/client/c/usage/connect).

### Initializing the Key

The application identifies records in the database using a key. In this example, the value for _key_ is the string _test-key_, to delete from namespace _test_ within the set _test-set_.

```cpp
as_key key;
as_key_init_str(&key, "test", "test-set", "test-key");
```

### Deleting Records

To delete the record from the database:

```cpp
if (aerospike_key_remove(&as, &err, NULL, &key) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

