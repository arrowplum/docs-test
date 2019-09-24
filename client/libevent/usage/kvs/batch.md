---
title: Batch Read Records
description: Use the Aerospike libevent client API to retrieve multiple records in a single transaction from an Aerospike database.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent client `ev2citrusleaf_get_many_digest()` API to retrieve multiple records in a single transaction.

### Non-blocking Calls

`ev2citrusleaf_get_many_digest()` is a non-blocking call. Any data retrieved returns through a client callback. 

To read 1000 records previously written to the database with digests known by the application:

```cpp
int result = ev2citrusleaf_get_many_digest(
		cluster,         // cluster object
		my_namespace,    // namespace name
		digests,         // batch of digests specifying which records to get
		1000,            // number of digests in batch
		NULL,            // bin name filter - NULL means get all
		0,               // number of bin names
		500,             // transaction timeout (milliseconds)
		my_batch_get_cb, // completion callback for this transaction
		NULL,            // callback returns this as udata - NULL in this example
		my_event_base);  // event base for this transaction
```

- The caller must specify a transaction timeout in milliseconds. If the transaction reaches this time limit, it is cut short. 
- The caller must specify a callback function and event base for the client callback return.

Parameters such as the digest array and bin names, are copied as necessary within the non-blocking function call, so they only need to last until the call is made. 

This call should never fail; however, it will fail if the parameters are blatantly illegal.

### Completion Callback

When transactions complete, the client makes a completion callback. A single callback is made per non-blocking call. All records retrieved  return in that callback.

On timeout, errors handling the transaction on the server and partial results return in a callback.

The callback signature is:

```cpp
typedef void (*ev2citrusleaf_get_many_cb)(int result, ev2citrusleaf_rec *recs,
		int n_recs, void *udata);
```

The callback provides a result code and array of individual record results.

`result` contains the overall result, which could be `EV2CITRUSLEAF_OK` even though individual record results are `EV2CITRUSLEAF_FAIL_NOTFOUND`. Typically, `result` is not `EV2CITRUSLEAF_OK` when the batch job times out or one or more cluster node transactions fail.

Partial record results can return on failure, in which case `n_recs` may be less than `n_digests` requested in the non-blocking call.

Note that the order of records in the `recs` array does not necessarily correspond to the order of the digests in the non-blocking call.

The client frees the `recs` array after the callback is made.

The array of individual record results is an array of `ev2citrusleaf_rec` objects.

The `ev2citrusleaf_rec` object is defined as follows:

```cpp
typedef struct ev2citrusleaf_rec_s {
	int               result;     // result for this record
	cf_digest         digest;     // digest identifying record
	uint32_t          generation; // record generation
	uint32_t          expiration; // record expiration, seconds from now
	ev2citrusleaf_bin *bins;      // record data - array of bins
	int               n_bins;     // number of bins in bins array
} ev2citrusleaf_rec;

```

An individual record `result` is either `EV2CITRUSLEAF_OK` or `EV2CITRUSLEAF_FAIL_NOTFOUND`.

- If result is `EV2CITRUSLEAF_OK`, bin data is present. 

The bin data returned for a record is just like that returned in single-record read transactions.

As with single-record read transactions, the application is responsible for freeing bin objects using `ev2citrusleaf_bins_free()`, but the client will free the `bins` array.

```cpp
// Forward declaration.
void handle_data(ev2citrusleaf_rec* rec);

void
my_batch_get_cb(int result, ev2citrusleaf_rec *recs, int n_recs, void *udata)
{
	switch (result) {
	case EV2CITRUSLEAF_OK:
		fprintf(stderr, "batch get transaction succeeded\n");
		break;
	case EV2CITRUSLEAF_FAIL_TIMEOUT:
		fprintf(stderr, "batch get transaction timed out\n");
		break;
	default:
		fprintf(stderr, "batch get result %d - partial record results", result);
		break;
	}

	// Examine the result - may have partial record results for "failure" cases.

	int n_found = 0;
	int n_not_found = 0;

	for (int i = 0; i < n_recs; i++) {
		switch (recs[i].result) {
		case EV2CITRUSLEAF_OK:
			n_found++;
			handle_data(&recs[i]);
			break;
		case EV2CITRUSLEAF_FAIL_NOTFOUND:
			n_not_found++;
			// No bin data should ever be returned here.
			break;
		default:
			// Individual records' result should be either OK or NOTFOUND.
			fprintf(stderr, "unexpected record result %d", recs[i].result);
			// No bin data should ever be returned here.
			break;
		}
	}
}

void
handle_data(ev2citrusleaf_rec* rec)
{
	// A shorthand partial representation of the digest for easy printing.
	uint64_t d = *(uint64_t*)&rec->digest;

	// Handle record data - here we just print some metadata.
	fprintf(stderr, "digest %lx - got %d bins:\n", d, p_rec->n_bins);

	for (int i = 0; i < p_rec->n_bins; i++) {
		fprintf(stderr, "  bin %s: data type %d, data size %lu\n",
				p_rec->bins[i].bin_name, p_rec->bins[i].object.type,
				p_rec->bins[i].object.size);
	}

	...

	// Free any allocated data.
	ev2citrusleaf_bins_free(p_rec->bins, p_rec->n_bins);
}

```

### Existence Checking

The `ev2citrusleaf_exists_many_digest()` batch operation is similar to `ev2citrusleaf_get_many_digest()`, except that no bin data is retrieved for records. `ev2citrusleaf_exists_many_digest()` checks for the existence of records in the database without the performance penalty of reading record data and returning it to the client.

```cpp
int result = ev2citrusleaf_exists_many_digest(
		cluster,            // cluster object
		my_namespace,       // namespace name
		digests,            // batch of digests specifying which records to get
		1000,               // number of digests in batch
		500,                // transaction timeout (milliseconds)
		my_batch_exists_cb, // completion callback for this transaction
		NULL,               // callback returns this as udata - NULL in this example
		my_event_base);     // event base for this transaction
```

`ev2citrusleaf_exists_many_digest()` uses the same callback mechanism as `ev2citrusleaf_get_many_digest()`, except that in the `ev2citrusleaf_rec` return objects, `bins` is always NULL and `n_bins` is always 0.

