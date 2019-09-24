---
title: Write Records
description: Use the Aerospike C client API write operations to store data in the Aerospike database.
---

The Aerospike C client API provides these operations to store data in the database:

- `aerospike_key_put()` &mdash; Write all the bins specified in a record to the cluster.
- `aerospike_key_operate()` &mdash; Perform operations on a record in the cluster, including writing and modifying bins.

This page discusses the `aerospike_key_put()` operation. The `aerospike_key_operate()` operation is discussed in [Record Operations](/docs/client/c/usage/kvs/multiops.html).

These examples are excerpted from _examples/basic\_examples/get_ in the Aerospike C client package.

To continue, the client must have an [established the cluster connection](/docs/client/c/usage/connect).

### Initializing Record Data

`aerospike_key_put()` requires an initialized record that contains the bins to store in the cluster. This example initializes a record on the stack with two bins: _test-bin-1_ (integer) and _test-bin-2_ (string):

```cpp
as_record rec;
as_record_inita(&rec, 2);
as_record_set_int64(&rec, "test-bin-1", 1234);
as_record_set_str(&rec, "test-bin-2", "test-bin-2-data");
```

### Initializing the Key

The application identifies a record in the database using a key. In the following example, the _key_ value is the string _test-key_, to store in the namespace _test_ within the set _test-set_. Other data types such as integer or blob are allowed for _key_. 

```cpp
as_key key;
as_key_init_str(&key, "test", "test-set", "test-key");
```

### Writing to the Database

Using the key write the record to the database:

```cpp
if (aerospike_key_put(&as, &err, NULL, &key, rec) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

### Cleaning Up Resources

Use `as_record_destroy()` to release records and their associated resources:

```cpp
as_record_destroy(rec);
```

In this example, the record and its associated bin and values are all allocated on the stack, so `as_record_destroy()` is not neccessarily required.

### Modifying Write Behavior

The default behavior of `put()` is:

- Create the record in the cluster (if it does not exist).
- Update bin values in the record (if the bins already exist). 
- Add bins (if they do not exist).
- If the record has other bins, retain them with the record.

Use `as_policy_write` in `aerospike_key_put()` to modify write behavior. These policy changes are allowed:

- Only write if the record already exists: 
-- change `write_policy.exist` to `AS_POLICY_EXISTS_CREATE`
- Replace a record _only_ if it exists: 
-- change `write_policy.exist` to `AS_POLICY_EXISTS_REPLACE`

#### Controlling the Write Transaction Timeout

When the application must respond to a caller during a specified time, set a transaction timeout on `aerospike_key_put()` by changing `write_policy.timeout` to the appropriate millesecond.

#### Read-Modify-Write

It is common to do a Read-Modify-Write of a record (or Check-and-Set). This involves:

- Reading the record.
- Modifying the record at the application level.
- Writing the modified data with the generation-previous read.

See the examples in _examples/basic\_examples/generation_ in the Aerospike C client package.

#### Changing Time-to-Live (TTL)

It is possible to specify a time-to-live value for a record on a write, by setting the `record.ttl` field:

- `ttl` = 0 &mdash; Use the default TTL in the namespace.
- `ttl` = -1 &mdash; The record never expires.
- Any other integer indicates the actual record TTL in seconds.

See the examples in _examples/basic\_examples/expire_ in the Aerospike C client package.
