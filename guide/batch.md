---
title: Batch
description: Use Aerospike batch requests to quickly and efficiently retrieve a large number of records. 
assets: /docs/guide/assets
---

Use Aerospike batch requests to quickly and efficiently retrieve a large number of records. 

Batch requests differ from queries because they are based on the primary key. The application passes a list of primary keys in the batch request. A series of results return.

Use batch requests to:
- efficiently implement a client-side join.
- store and then retrieve multiple data points in a calculation.
- determine key expiration or if keys still exist.

Aerospike supports these batch requests:
- `batchGet`
- `batchExists`
- `batchGetHeaders`: Read record metadata (expiration/TTL and generation) only. Do not read bins.
- `batchGetComplex`: Read records with varying namespaces, bin names, and read types in one batch.

{{#note}}
`batchGetComplex` requires Aerospike Server version 3.6.0 or later.
{{/note}}

**Example**

The following Java code example reads multiple records in a single batch call.

```java
	void batchRead (
		AerospikeClient client,
		BatchPolicy policy,
		String namespace,
		String set,
		String binName,
		int size
	) throws Exception {
		Key[] keys = new Key[size];
		for (int i = 0; i < size; i++) {
			keys[i] = new Key(namespace, set, i + 1);
		}

		Record[] records = client.get(policy, keys, binName);

		for (int i = 0; i < records.length; i++) {
			Key key = keys[i];
			Record record = records[i];
			
			if (record != null) {
				Object value = record.getValue(binName);
				// Process value here.
			}
        }
    }
```

## Implementation

The client determines which records are where, and creates a batch request to each server. The batch request lists the primary keys to retrieve and optional bin names. Batch requests occur in series or parallel, depending on the batch policy. Parallel requests in synchronous mode require extra threads, which are created or taken from a thread pool.

Batch requests use a single network transaction to each server, which saves the developer from having to parallelize requests. Multiple keys use one network request, which is very beneficial for a large number of small records, but not as beneficial when the number of records per server is small or the data returned per record is large. Note that batch requests can increase the latency of some of requests, but only because clients normally wait until all keys are retrieved from the server nodes before it returns control to the caller.

Some clients such as the C client support delivering each record as soon as it arrives, allowing client applications to process data from the fastest server first.

## Batch Protocols

Starting with version 3.6.0, the Aerospike Server supports two batch protocols: Batch Direct and Batch Index.

### Batch Index

The batch index allows a single request to retrieve a record using any combination of keys, namespaces, bin name filters, and read types (read bins, read header, exists). The server handles this protocol by distributing keys in the batch on the single record transaction queues and threads. Since batch index uses the same transaction path as single record reads, proxies to other servers are supported during cluster migration. Batch requests also run at the same priority level as single record transactions.

The batch index process is:

1. Receive a batch request from the client.
1. Split the request into multiple single record requests.
1. Distribute these single record requests across the single record thread queues.
1. Consolidate individual responses into 128KB response buffers.
1. Place response buffers on the batch response thread queue so that the transaction thread does not block.
1. Batch response threads return these response buffers to the client.

If the connected server does not support batch index, the batch direct protocol is used.

The server provides the following configuration variables for batch index performance tuning.

| Name                       | Default | Max | Dynamic | Description |
| -------------------------- | -------:|:---:| ------- | ----------- |
| [`batch-max-requests`](/docs/reference/configuration/#batch-max-requests)         |    5000 |     | true    | Maximum number of keys allowed per node. Used to prevent unexpectedly large batch requests from causing server instability due to excessive memory consumption. |
| [`batch-max-buffers-per-queue`](/docs/reference/configuration/#batch-max-buffers-per-queue) | 	   255 |     | true    | Maximum number of 128KB response buffers allowed in each batch index queue. If all batch index queues are full, new batch requests are rejected. |
| [`batch-max-unused-buffers`](/docs/reference/configuration/#batch-max-unused-buffers)   | 	   256 |     | true    | Maximum number of 128KB response buffers allowed in the buffer pool. If the limit is reached, new buffers will be created (if within the batch-max-buffer-per-queue) and destroyed when completed at the end of the batch request. This is basically the size of the buffer pool. |
| [`batch-index-threads`](/docs/reference/configuration/#batch-index-threads)        |       4 |  64 | true    | Number of batch response worker threads. Each thread has its own queue. These threads only handle returning batch response buffers to the client using sockets. The buffers are created in single record transaction threads. The maximum memory used can be computed as: `batch-index-threads` x `batch-max-buffer-per-queue` x 128KB. |

{{#note}}
The client can change dynamic variables with the server running.
{{/note}}

The server also provides the following batch index statistics variables (refer to the [full metrics reference manual](/docs/reference/metrics)):

| Name                            | Description |
| ------------------------------- | ----------- | 
| [`batch_index_initiate`](/docs/reference/metrics/#batch_index_initiate)          | Number of batch index requests received. |
| [`batch_index_queue`](/docs/reference/metrics/#batch_index_queue)             | Number of batch index requests and response buffers remaining on each batch queue.  Format: &lt;q1 requests&gt; :&lt;q1 buffers&gt; ,&lt;q2 requests&gt; :&lt;q2 buffers&gt;,...
| [`batch_index_complete`](/docs/reference/metrics/#batch_index_complete)          | Number of completed batch index requests. |
| [`batch_index_timeout`](/docs/reference/metrics/#batch_index_timeout)           | Number of timed-out batch index requests. |
| [`batch_index_error`](/docs/reference/metrics/#batch_index_error)            | Number of batch index requests rejected because of errors. |
| [`batch_index_unused_buffers`](/docs/reference/metrics/#batch_index_unused_buffers)    | Number of available 128KB response buffers in the buffer pool. |
| [`batch_index_huge_buffers`](/docs/reference/metrics/#batch_index_huge_buffers)      | Number temporary response buffers created that exceeded 128KB.  Huge buffers are created when one of the records is retrieved that is greater than 128KB.  Huge records do not benefit from batching and can result in excessive memory thrashing on the server.  |
| [`batch_index_created_buffers`](/docs/reference/metrics/#batch_index_created_buffers)   | Number of 128KB response buffers created.  Response buffers are created when there are no buffers left in the pool.  If this number consistently increases and there is available memory, then `batch-max-unused-buffers` should be increased. |
| [`batch_index_destroyed_buffers`](/docs/reference/metrics/#batch_index_destroyed_buffers) | Number of 128KB response buffers destroyed.  Response buffers are destroyed when there is no slot left to put the buffer back into the pool.  The maximum response buffer pool size is `batch-max-unused-buffers`. |
| [`batch-index`](/docs/operations/monitor/latency/index.html#batch-index)             | Batch index performance histogram. |


And for batch index sub transactions: 

| Name                            | Description |
| ------------------------------- | ----------- |
| [`batch_sub_proxy_complete`](/docs/reference/metrics/#batch_sub_proxy_complete)      | Number of proxied batch-index sub transactions that completed. |
| [`batch_sub_proxy_error`](/docs/reference/metrics/#batch_sub_proxy_error)         | Number of proxied batch-index sub transactions that failed with an error. |
| [`batch_sub_proxy_timeout`](/docs/reference/metrics/#batch_sub_proxy_timeout)       | Number of proxied batch-index sub transactions that timed out. |
| [`batch_sub_read_error`](/docs/reference/metrics/#batch_sub_read_error)          | Number of batch-index read sub transaction that failed with an error. |
| [`batch_sub_read_not_found`](/docs/reference/metrics/#batch_sub_read_not_found)      | Number of batch-index read sub transaction that resulted in not found. |
| [`batch_sub_read_success`](/docs/reference/metrics/#batch_sub_read_success)        | Number of successful batch-index read sub transactions. |
| [`batch_sub_read_timeout`](/docs/reference/metrics/#batch_sub_read_timeout)        | Number of batch-index read sub transactions that timed out. |
| [`batch_sub_tsvc_error`](/docs/reference/metrics/#batch_sub_tsvc_error)          | Number of batch-index read sub transactions that failed with an error in the transaction service, before attempting to handle the transaction.  For example protocol errors or security permission mismatch. |
| [`batch_sub_tsvc_timeout`](/docs/reference/metrics/#batch_sub_tsvc_timeout)        | Number of batch-index read sub transactions that timed out in the transaction service, before attempting to handle the transaction.  For example protocol errors or security permission mismatch. |
| [`retransmit_all_sub_dup_res`](/docs/reference/metrics/index.html#retransmit_all_sub_dup_res)  | Number of retransmits that occurred during batch sub transactions that were being duplicate resolved. Note this includes retransmits originating on the client as well as proxying nodes. |
| [`retransmit_batch_sub_dup_res`](/docs/reference/metrics/index.html?show-removed=1#retransmit_batch_sub_dup_res)  | Number of retransmits that occurred during batch sub transactions that were being duplicate resolved. Replaced with retransmit_all_batch_sub_dup_res as of version 4.5.1.5. |
| [`early_tsvc_batch_sub_error`](/docs/reference/metrics/#early_tsvc_batch_sub_error)    | Number of errors early in the transaction for batch sub transactions.  For example, bad/unknown namespace name or security authentication errors. |

Finally, refer to the [latency monitoring](/docs/operations/monitor/latency/index.html#batch-sub-start) page for latency histograms of batch index and its sub transactions.

### Batch Direct (legacy batch protocol supported on all server versions prior to 4.4)

The batch direct protocol allows a single request to retrieve records from multiple keys using a single namespace and bin name filter. The server handles this protocol with separate batch queue/threads and single record transaction queues/threads, which places batch requests at a lower priority than single record transactions.

Please note that the batch direct protocol is not supported on server versions 4.4 or later.

 The batch direct process is:
1. Receive the batch request from the client.
1. Put the entire batch request in the batch thread queue.
1. The batch threads process the entire batch request, including low-level reads and sending the results to the client.

Aerospike implements batch direct protocols with direct low-level database read methods. Batch direct is faster when all keys are in the same namespace; however, there is one important drawback: Batch direct does not proxy to a different server node on record not found errors. This can happen after a node is added or removed from the cluster and there is a lag between the records being migrated and the client partition map update (once per second).

Legacy clients continue to use the batch direct protocol. Newer clients default to batch index when available. To choose batch direct in your client, change the batch policy variable. For example, in the Aerospike Java client, enable the `BatchPolicy.useBatchDirect` batch policy variable.

The server provides the following configuration batch direct performance tuning variables. 

| Name                 | Default | Max | Dynamic | Description |
| -------------------- | -------:|:---:| ------- | ----------- | 
| `batch-max-requests`   |    5000 |  &ndash;  | true    | Maximum number of keys allowed per node. Prevents unexpectedly large batch requests from causing server instability due to excessive memory consumption. |
| `batch-priority`       |     200 |  &ndash;  | true    | Number of sequential reads before yielding. The higher the number, the higher the priority. |
| `batch-threads`        |       4 |  64 | true    | Number of batch direct worker threads, which process the full batch request. There is only one batch queue for all batch threads. |


{{#note}}
The client can change dynamic variables with the server running.
{{/note}}

The server also provides the following batch direct statistics variables:

| Name             | Description |
| ---------------- | ----------- | 
| `batch_initiate`   | Number of batch direct requests received. |
| `batch_queue`      | Number of batch direct requests remaining in the queue awaiting processing. |
| `batch_tree_count` | Number of tree lookups for all batch direct requests. |
| `batch_timeout`    | Number of timed-out batch direct requests. |
| `batch_errors`    | Number of batch direct requests rejected for errors. |
| `batch_q_process`  | Batch direct performance histogram. |

## Known Limitations
- Batch writes are not supported.

## References

See these topics for language-specific examples:

- [Java](/docs/client/java/examples/application/batch.html)
- [C# .NET](/docs/client/csharp/examples/application/batch.html)
- [C](/docs/client/c/usage/kvs/batch.html)
- [Node.js](/docs/client/nodejs/usage/kvs/read.html)
- [Go](/docs/client/go/usage/kvs/batch.html)
- [Python](/docs/client/python/usage/kvs/read.html#running-batch-operations)
- [PHP](/docs/client/php/usage/kvs/read.html#batch-operations)
- [Ruby](/docs/client/ruby/usage/kvs/batch.html)
