---
title: Reading Records
description: Use the Aerospike libevent client APIs to retrieve data from an Aerospike database.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent client APIs to retrieve data from an Aerospike database.  `ev2citrusleaf_get()` allows an application to retrieve data from an Aerospike database cluster.

### Non-blocking Calls

This non-blocking call retrieves data from a record, and returns using the client callback. `ev2citrusleaf_get()` allows the application to read specific bins of a record.

To read two bins of the record written in [Writing a Record](/docs/client/libevent/usage/kvs/write.html):

```cpp
// Specify which bins to get.
const char* bin_names[] = { "test-bin-B", "test-bin-C" };

int result = ev2citrusleaf_get(
		cluster,        // cluster object
		my_namespace,   // namespace name
		my_set,         // set name
		&key,           // key of record to get
		bin_names,      // names of bins to get
		2,              // get two (of the three) bins
		100,            // transaction timeout (milliseconds)
		my_client_cb,   // completion callback for this transaction
		NULL,           // callback returns this as udata - NULL in this example
		my_event_base); // event base for this transaction
```

- The caller must specify a transaction timeout in milliseconds. If the transaction reaches this time limit, it is cut short. 
- The caller must specify a callback function and event base for the client callback return.

Parameters such as the digest array and bin names, are copied as necessary within the non-blocking function call, so they only need to last until the call is made. 

This call should never fail; however, it will fail if the parameters are blatantly illegal.

### Read All Bins of a Record

To read all bins of a record using `ev2citrusleaf_get_all()`:

```cpp
// Get all the bins.
int result = ev2citrusleaf_get_all(cluster, my_namespace, my_set, &key, 100,
		my_client_cb, NULL, my_event_base);
```

This is the same as `ev2citrusleaf_get()`, but doesn't accept the `bin_names` and `n_bins` parameters.


### Completion Callback

When transactions complete, the client makes a completion callback. A single callback is made per non-blocking call. All records retrieved  return in that callback.

On timeout, errors handling the transaction on the server and partial results return in a callback.

The callback signature is:

```cpp
typedef void (*ev2citrusleaf_callback)(int return_value,
		ev2citrusleaf_bin *bins, int n_bins, uint32_t generation,
		uint32_t expiration, void *udata);
```

`return_value` in the callback indicates transaction success. On success, if `return_value` is `EV2CITRUSLEAF_OK`, the callback provides the requested record data.

```cpp
void
my_client_cb(int return_value, ev2citrusleaf_bin* bins, int n_bins,
		uint32_t generation, uint32_t expiration, void* udata)
{
	switch (return_value) {
	case EV2CITRUSLEAF_OK:
		// Handle record data.
		break;
	case EV2CITRUSLEAF_FAIL_TIMEOUT:
		fprintf(stderr, "read transaction timed out\n");
		return;
	case EV2CITRUSLEAF_FAIL_NOTFOUND:
		fprintf(stderr, "read transaction did not find record\n");
		return;
	default:
		fprintf(stderr, "read transaction failed, err: %d\n", return_value);
		return;
	}

	// Handle record data - here we just print some metadata.
	fprintf(stderr, "success - got %d bins:\n", n_bins);

	for (int i = 0; i < n_bins; i++) {
		fprintf(stderr, "  bin %s: data type %d, data size %lu\n",
				bins[i].bin_name, bins[i].object.type, bins[i].object.size);
	}

	...

	// Free any allocated data.
	ev2citrusleaf_bins_free(bins, n_bins);
}
```

The record data retrieved is provided in an array of `ev2citrusleaf_bin` objects. Each `ev2citrusleaf_bin` object contains the bin name and data as a `ev2citrusleaf_object` object.

Bins can appear in any order in the array. To determine a specific bin, the application must check all bin names.

The application must free bin objects using `ev2citrusleaf_bins_free()`, but the client frees the `bins` array. The application must free any allocated bin data (after consuming it) within the scope of the callback by calling `ev2citrusleaf_bins_free()`. This frees the data object in each bin (if need be). The application should copy bin data if it is needed beyond the scope of the callback.

### Digest-only Calls

Use `ev2citrusleaf_get_digest()` or `ev2citrusleaf_get_all_digest()` to read a record using the digest hash instead of the key for the record ID:

```cpp
cf_digest digest;
ev2citrusleaf_calculate_digest(my_set, &key, &digest);

// Some time later ...
int result = ev2citrusleaf_get_all_digest(cluster, my_namespace, &digest, 100,
		my_client_cb, NULL, my_event_base);
```

