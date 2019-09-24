---
title: Read a Record
description: Use Aerospike Java client APIs to read records from the Aerospike Database.
---

To read a record from the database, the Aerospike Java client library can:

- Read all bins in a record.
- Read specified bins in a record.
- Read only the metadata of a record.

Each method returns a `Record` object that contains the metadata and bins of the record.

To read multiple records in the same call, see [Batch Reads](/docs/client/java/usage/kvs/batch.html) section.

### Read All Bins in a Record

This example reads all bins for a record:

```java
Record record = client.get(policy, key);
```

### Read Specific Bins of a Record

This example only reads two bins from the record: _name_ and _age_:

```java
Record record = client.get(policy, key, "name", "age");
```
