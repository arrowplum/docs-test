---
title: Read Records
description: Use the Aerospike C client API to read a record from the Aerospike database.
---

The Aerospike C client API has the following operations for reading a record from the database:

- `aerospike_key_get()` &mdash; Read all the bins of a record.
- `aerospike_key_select()` &mdash; Read the specified bins of a record.
- `aerospike_key_exists()` &mdash; Check for the existence of a record.
- `aerospike_key_operate()` &mdash; Perform operations on a record, including a read operation on specific bins.

This page discusses the first three operations. The `aerospike_key_operate()` operation is discussed in [Record Operations](multiops.html).

These examples are excerpted from _examples/basic\_examples/get_ in the Aerospike C client package.

To continue, the client must have an [established the cluster connection](/docs/client/c/usage/connect).

### Initializing the Key

The application identifies a record in the database using a key. In the following example, the _key_ value is the string _test-key_, stored in the namespace _test_ within the set _test-set_. Other data types such as integer or blob are allowed for _key_. 

```cpp
as_key key;
as_key_init_str(&key, "test", "test-set", "test-key");
```

### Reading All Bins in a Record

Use `aerospike_key_get()` to read all bins in a record. This operation populates the `record` parameter with as many bins as possible. In the example, `as_record` (line 1) is initialized as a NULL pointer to allow the read operation to allocate a new record on the heap with enough bins to hold all bins returned from the database.

```cpp
as_record* p_rec = NULL;
  
if (aerospike_key_get(&as, &err, NULL, &key, &p_rec) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
 
as_record_destroy(p_rec);
```

You can use `aerospike_destroy()` to destroy the client and release all of its resources:

```cpp
aerospike_destroy(&as);
```


### Reading the Specific Bins in a Record

To conserve resources, instead of fetching the entire record, specify the exact bins to fetch. To to this, first build a a NULL-terminated array of strings, where each string is a bin name, and then call `aerospike_key_select()`.

This example shows an array of two bins to read from the database: _test-bin-1_ and _test-bin-3_. The array is NULL-terminated, to tell the read operation that there are no more bin names. 

```cpp
as_record* p_rec = NULL;
static const char* bins_1_3[] = { "test-bin-1", "test-bin-3", NULL };
  
if (aerospike_key_select(&as, &err, NULL, &key, bins_1_3, &p_rec) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
 
as_record_destroy(rec);
```

Use `aerospike_destroy()` to destroy the client and release all of its resources.

### Traversing Bins

See [Traversing Bins of a Record](/docs/client/c/best_practices/records.html#TraverseBin).

### Determining the Existence of a Record

Use `aerospike_key_exists()` to populate the record with metadata such as generation and time-to-live. This allows the application to determine if a record exists in the database. 

```cpp
as_record* p_rec = NULL;
 
if (aerospike_key_exists(&as, &err, NULL, &key, &p_rec) == AEROSPIKE_ERR_RECORD_NOT_FOUND) {
    fprintf(stderr, "record not found");
} 

```

### Controlling the Read Transaction Timeout
 
When an application must respond to a caller during a specified time, set a transaction timeout on the corresponding read by changing `read_policy.timeout` to the appropriate millesecond.


