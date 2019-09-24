---
title: Record Operations
description: Use the Aerospike libevent client to modify and read multiple records in a single transaction.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent client to modify and read multiple records in a single transaction (that is, _atomic_ modification).

The Aerospike libevent client API can perform the following operations on a record:

Operation | Description | Conditions
--- | --- | ---
write | Write a value to a bin. |
read | Read the value of a bin. |
add | Add an integer to the existing value of a bin. | Existing value must be integer.

{{#note}} 
Read operations are performed after all other operations complete. The order of the non-read operations is as defined by the application.
{{/note}}

{{#imp}}
Although the Aerospike database server supports the append, prepend, and touch operations, they are not supported in this client.
{{/imp}}

### Operation Specification

Use `operate` calls to specify the operations for a transaction in `ev2citrusleaf_operation` objects. `ev2citrusleaf_operation` are like `ev2citrusleaf_bin` objects with a bin name and data as a `ev2citrusleaf_object` object, but also have an operation type:

```cpp
enum ev2citrusleaf_operation_type { CL_OP_WRITE, CL_OP_READ, CL_OP_ADD };
```

To specify the operations to increment two bins of the record written in [Writing a Record](/docs/client/libevent/usage/kvs/write.html) and return the results:

```cpp
ev2citrusleaf_operation ops[4];

// Increment bin B by 1.
strcpy(ops[0].bin_name, "test-bin-B");
ops[0].op = CL_OP_ADD;
ev2citrusleaf_object_init_int(&ops[0].object, 1);

// Increment bin C by 2.
strcpy(ops[1].bin_name, "test-bin-C");
ops[1].op = CL_OP_ADD;
ev2citrusleaf_object_init_int(&ops[1].object, 2);

// Read bin B.
strcpy(ops[2].bin_name, "test-bin-B");
ops[2].op = CL_OP_READ;
ev2citrusleaf_object_set_null(&ops[2].object);

// Read bin C.
strcpy(ops[3].bin_name, "test-bin-C");
ops[3].op = CL_OP_READ;
ev2citrusleaf_object_set_null(&ops[3].object);
```

### Non-blocking Calls

`ev2citrusleaf_operation` is a non-blocking call. Any bins read return using the client callback. If the specified record does not exist, any `write` or `add` operation creates the record. If the specified bin does not exist, a `write` or `add` operation creates the bin. 

When an `add` operation creates a bin, the bin has an integer value set to the increment the value specified by the operation.

```cpp
// Make the non-blocking operate call.
int result = ev2citrusleaf_operate(
		cluster,        // cluster object
		my_namespace,   // namespace name
		my_set,         // set name
		&key,           // key of record to write
		ops,            // ops (array) to perform
		4,              // four ops for this transaction
		NULL,           // write parameters - NULL for defaults
		100,            // transaction timeout (milliseconds)
		my_client_cb,   // completion callback for this transaction
		NULL,           // callback returns this as udata - NULL in this example
		my_event_base); // event base for this transaction
```

- The caller must specify the transaction timeout in milliseconds. If the transaction reaches this limit, it is cut short.
- The caller mus specify a callback function and event base for the callback.

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

`return_value` in the callback indicates transaction success. 

If `return_value` is `EV2CITRUSLEAF_OK`, the callback contains requested bin data.

The application must free bin objects using `ev2citrusleaf_bins_free()`, but the client frees the `bins` array:

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
		fprintf(stderr, "operate transaction timed out\n");
		return;
	default:
		fprintf(stderr, "operate transaction failed, err: %d\n", return_value);
		return;
	}

	// Handle record data - here we just print the resulting integer values.
	fprintf(stderr, "success - got %d bins:\n", n_bins);

	for (int i = 0; i < n_bins; i++) {
		if (bins[i].object.type != CL_INT) {
			fprintf(stderr, "  bin %s: data type %d - should be integer!\n",
					bins[i].bin_name, bins[i].object.type);
		}
		else {
			fprintf(stderr, "  bin %s: value after incrementing is %ld\n",
					bins[i].bin_name, bins[i].object.u.i64);
		}
	}

	...

	// Free any allocated data - unnecessary for integer data, but good form.
	ev2citrusleaf_bins_free(bins, n_bins);
}
```

### Digest-only Calls

Use `ev2citrusleaf_operate_digest()` to operate on a record using the digest hash instead of the key for the record ID:

```cpp
cf_digest digest;
ev2citrusleaf_calculate_digest(my_set, &key, &digest);

// Some time later ...
int result = ev2citrusleaf_operate_digest(cluster, my_namespace, &digest, ops, 4,
		NULL, 100, my_client_cb, NULL, my_event_base);
```

{{#note}}
Use `ev2citrusleaf_operate_digest()`cautiously. `ev2citrusleaf_operate_digest()` lacks a `set` parameter, so **never use it to create records in the database**. Only use `ev2citrusleaf_operate_digest()` to modify existing records in a known set.
{{/note}}

