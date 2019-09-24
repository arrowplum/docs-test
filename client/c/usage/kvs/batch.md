---
title: Batch Read Records
description: Run batch record reads using the Aerospike C client on the Aerospike database.
categories:
  - aerospike-client-c
tags:
  - aerospike-client-c
  - c
---

Use the following calls to perform multiple record reads from the cluster in one transaction:

- `aerospike_batch_get()` &mdash; Return all bins of the specified records.
- `aerospike_batch_exists()` &mdash; Return the metadata (such as, time-to-live and generation) for the specified records.

These examples are in _examples/batch_examples/get_ in the Aerospike C client package.

To continue, the client must have an [established the cluster connection](/docs/client/c/usage/connect).

### Initializing a Batch Request

Before making a batch call, intialize an `as_batch` object. This example initializes `as_batch` with 1000 records to fetch:

```cpp
as_batch batch;
as_batch_inita(&batch, 1000);
```

### Initializing Keys to Read

To initialize the keys to read:

```cpp
for (uint32_t i = 0; i < 1000; i++) {
  as_key_init_int64(as_batch_keyat(&batch, i), "test", "demoset", (int64_t)i);
}
```

This call sets up the 1000 keys using `as_batch_keyat()`. The key data type is integer, in the range 0 to 1000. Different key data types such as string are allowed. The keys are made for the namespace _test_ within set _demoset_.

{{#note}}
The keys must all belong to the same namespace.
{{/note}}

### Reading Records

To retrieve records: 

```cpp
if (aerospike_batch_get(&as, &err, NULL, &batch, batch_read_cb, NULL) 
      != AEROSPIKE_OK) {
    LOG("aerospike_batch_get() returned %d - %s", err.code, err.message);
    cleanup(&as);
    exit(-1);
}
```

This call groups keys based on which Aerospike server node can best handle the request and initiate one batch job for each node in the cluster. There are six `NUM_BATCH_THREADS` batch worker threads that process all batch jobs for all nodes. 

{{#note}}
If your cluster has more than six nodes, increase the default thread count to allow full node-level parallelism.
{{/note}}

### Processing the Results

In the previous example, `aerospike_batch_get()` takes a `batch_read_cb` callback function as a parameter with the following type:

```cpp
bool (* aerospike_batch_read_callback)(const as_batch_read * results, uint32_t n, void * udata);
```

This callback function is invoked after all records return from all the nodes, and in the same order as the keys are passed.

An application can examine the list `as_batch_read` results and process each record. These records and values are only valid within the scope of the callback, and must explicitly be copied if needed outside the callback scope.

```cpp
bool
batch_read_cb(const as_batch_read* results, uint32_t n, void* udata)
{

  uint32_t n_found = 0;

  for (uint32_t i = 0; i < n; i++) {
    LOG("index %u, key %" PRId64 ":", i,
        as_integer_getorelse((as_integer*)results[i].key->valuep, -1));

    if (results[i].result == AEROSPIKE_OK) {
      LOG("  AEROSPIKE_OK");
      n_found++;
    }
    else if (results[i].result == AEROSPIKE_ERR_RECORD_NOT_FOUND) {
      // The transaction succeeded but the record doesn't exist.
      LOG("  AEROSPIKE_ERR_RECORD_NOT_FOUND");
    }
    else {
      // The transaction didn't succeed.
      LOG("  error %d", results[i].result);
    }
  }

  return true;
}
```

To pass a global object during the callback, supply a `userdata` parameter in `aerospike_batch_get()`.

### Reading Record Metadata from Database

Instead of always reading the entire bin data of full records, the application can make an equivalent call to only return the metadata of the record (such as time-to-live and generation). This is preferred when the application only needs to determine if a record exists, and does not want to incur the cost of actually reading the data.

```cpp
if (aerospike_batch_exists(&as, &err, NULL, &batch, batch_read_cb, NULL) 
      != AEROSPIKE_OK) {
    LOG("aerospike_batch_exists() returned %d - %s", err.code, err.message);
    cleanup(&as);
    exit(-1);
}
```

### Cleaning Up Resources

Always clean up keys made with heap allocations.

