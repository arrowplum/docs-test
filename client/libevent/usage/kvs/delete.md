---
title: Deleting Records
description: Use the Aerospike libevent client API to remove data from an Aerospike Database.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent Client API `ev2citrusleaf_delete()` function to remove a record from an Aerospike database cluster.

### Non-blocking Calls

Use this non-blocking call to delete the record written in [Writing a Record](/docs/client/libevent/usage/kvs/write.html):

```cpp
// Make the non-blocking delete call.
int result = ev2citrusleaf_delete(
		cluster,        // cluster object
		my_namespace,   // namespace name
		my_set,         // set name
		&key,           // key of record to write
		NULL,           // write parameters - NULL for defaults
		100,            // transaction timeout (milliseconds)
		my_client_cb,   // completion callback for this transaction
		NULL,           // callback returns this as udata - NULL in this example
		my_event_base); // event base for this transaction
```

- The caller must specify a transaction timeout in milliseconds. If the transaction reaches this time limit, it is cut short. 
- The caller must specify a callback function and event base for the client callback return.

The caller should specify a transaction timeout in milliseconds — if the transaction reaches this time limit, it will be cut short.
The caller must also specify a callback function, and the event base on which the client will make that callback.

This call should never fail; however, it will fail if the parameters are blatantly illegal.

### Completion Callback

When transactions complete, the client makes a completion callback. A single callback is made per non-blocking call. All records retrieved  return in that callback.
A timeout, errors handling the transaction on the server, and partial results will also result in a callback.

The callback signature is:

```cpp
typedef void (*ev2citrusleaf_callback)(int return_value,
		ev2citrusleaf_bin *bins, int n_bins, uint32_t generation,
		uint32_t expiration, void *udata);
```

`return_value` in the callback indicates transaction success. For successful `ev2citrusleaf_delete()` calls, this result code is the only relevant parameter.

```cpp
void
my_client_cb(int return_value, ev2citrusleaf_bin* bins, int n_bins,
		uint32_t generation, uint32_t expiration, void* udata)
{
	switch (return_value) {
	case EV2CITRUSLEAF_OK:
		fprintf(stderr, "delete transaction succeeded\n");
		break;
	case EV2CITRUSLEAF_FAIL_TIMEOUT:
		fprintf(stderr, "delete transaction timed out\n");
		break;
	default:
		fprintf(stderr, "delete transaction failed, err: %d\n", return_value);
		break;
	}
}
```

### Digest-only Calls

`ev2citrusleaf_delete_digest()` is the Aerospike libevent client alternate delete method that uses the digest hash instead of the key to identify the record:

```cpp
cf_digest digest;
ev2citrusleaf_calculate_digest(my_set, &key, &digest);

// Some time later ...
int result = ev2citrusleaf_delete_digest(cluster, my_namespace, &digest, NULL,
		100, my_client_cb, NULL, my_event_base);
```

