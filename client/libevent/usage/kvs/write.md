---
title: Writing Records
description: Use the Aerospike libevent client APIs to store data in an Aerospike database.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent client `ev2citrusleaf_put()` API to store data in an Aerospike database. 

### Record Keys

Records are uniquely identified by the namespace, set, and key. Within specified namespaces and sets, each key must be unique. Keys used in the database transaction calls are `ev2citrusleaf_object` objects. The application must convert record keys to this form to use the Aerospike libevent client APIs.

For example, if the application uses string keys:

```cpp
// Initialize a record key.
ev2citrusleaf_object key;
ev2citrusleaf_object_init_str(&key, (char*)"test-key");
```

### Record Data

Record data is stored in one or more bins (maximum bin name length is 14 characters). Record data passes bin names to the Aerospike libevent client API as an array of `ev2citrusleaf_bin` objects, and passes data as a `ev2citrusleaf_object` object.

```cpp
// Write a record with three bins to the database.
ev2citrusleaf_bin bins[3];

// The first bin has a string value.
strcpy(bins[0].bin_name, "test-bin-A");
ev2citrusleaf_object_init_str(&bins[0].object, "test-value-A");

// The second bin has an integer value.
strcpy(bins[1].bin_name, "test-bin-B");
ev2citrusleaf_object_init_int(&bins[1].object, 12345);

// The third bin also has an integer value.
strcpy(bins[1].bin_name, "test-bin-C");
ev2citrusleaf_object_init_int(&bins[2].object, 67890);
```

### Writes do not Replace Whole Records

Write operations do not affect existing bins not specified in the non-blocking write call. For example, if a record exists with bins A, B, and C and the write operation specifies bins B and D, then the operation overwrites bin B and creates bin D, resulting in a record with bins A, B, C, and D.

{{#note}}
The Aerospike server does support _replace_ operations; however they are planned for future releases.
{{/note}}

### Non-blocking Calls

Once a record key and data are ready, the record can be written to the database cluster using the non-blocking call. The caller use write parameters (see `ev2citrusleaf_write_parameters` in `ev2citrusleaf.h`) to specify extra information (for example, the record time-to-live).

- The caller must specify a transaction timeout in milliseconds. If the transaction reaches this time limit, it is cut short. 
- The caller must specify a callback function and event base for the client callback return.

```cpp
// Make the non-blocking write call.
int result = ev2citrusleaf_put(
		cluster,        // cluster object
		my_namespace,   // namespace name
		my_set,         // set name
		&key,           // key of record to write
		bins,           // bins (array) to write
		3,              // three bins for this transaction
		NULL,           // write parameters - NULL for defaults
		100,            // transaction timeout (milliseconds)
		my_client_cb,   // completion callback for this transaction
		NULL,           // callback returns this as udata - NULL in this example
		my_event_base); // event base for this transaction
```

Parameters such as the digest array and bin names, are copied as necessary within the non-blocking function call, so they only need to last until the call is made. 

This call should never fail; however, it will fail if the parameters are blatantly illegal.

### Completion Callback

When transactions complete, the client makes a completion callback. A single callback is made per non-blocking call. All records retrieved  return in that callback.

On timeout, errors handling the transaction on the server and partial results return in a callback.

The callback signature is:

```cpp
typedef void (*ev2citrusleaf_callback)(int return_value,
		ev2citrusleaf_bin *bins, int n_bins, uint32_t generation,
		uint32_t expiration, void *udata);
```

`return_value` in the callback indicates transaction success or not. On success, the result code is the only relevant parameter.

```cpp
void
my_client_cb(int return_value, ev2citrusleaf_bin* bins, int n_bins,
		uint32_t generation, uint32_t expiration, void* udata)
{
	switch (return_value) {
	case EV2CITRUSLEAF_OK:
		fprintf(stderr, "write transaction succeeded\n");
		break;
	case EV2CITRUSLEAF_FAIL_TIMEOUT:
		fprintf(stderr, "write transaction timed out\n");
		break;
	default:
		fprintf(stderr, "write transaction failed, err: %d\n", return_value);
		break;
	}
}
```

### Digest-only Calls

Use `ev2citrusleaf_put_digest()` to allow the application to write a record using the digest hash instead of the key as the record ID.

```cpp
cf_digest digest;
ev2citrusleaf_calculate_digest(my_set, &key, &digest);

// Some time later ...
int result = ev2citrusleaf_put_digest(cluster, my_namespace, &digest, bins, 3,
		NULL, 100, my_client_cb, NULL, my_event_base);
```

{{#note}}
This function is deprecated. Use with caution. `ev2citrusleaf_put_digest()` lacks a `set` parameter, so **never use it to create records in the database**. Only use `ev2citrusleaf_put_digest()` to modify existing records in a known set.
{{/note}}

