---
title: Connecting
description: Use the Aerospike libevent client to connect to an Aerospike database cluster.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent client to connect to an Aerospike database cluster.

###Initializing

Ensure that the Aerospike libevent client is initialized, and create a `ev2citrusleaf_cluster` object:

```cpp
ev2citrusleaf_init(NULL);
cluster = ev2citrusleaf_cluster_create(mgr_base, NULL);
```

To allow the client to use an internally created event base for cluster management, pass a NULL event base in `create`. This creates the internal thread and immediately starts the event base loop. 

{{#note}}
The internal event base is not available to the application.
{{/note}}

###Specifying the Cluster

To specify a host (any node) in the database cluster:

```cpp
ev2citrusleaf_cluster_add_host(cluster, "127.0.0.1", 3000);

```
The Aerospike libevent client adds a socket event to the cluster management event base, which starts a transaction to retrieve cluster information. The transaction proceeds when the cluster management event base loop is running; it may already be running (such as when using the internal cluster management option) or be started by subsequent code.

###Waiting

The application can initiate database transactions without waiting for a stable cluster, which can cause early transactions to timeout depending on specified timeout and host response time. 

The application can wait until the client detects a stable cluster before initiating transactions. For example, if the cluster management event base loop is active in another thread, the application can just poll in a loop until it detects a stable cluster:

```cpp
bool wait_for_cluster(ev2citrusleaf_cluster *cluster)
{
	int tries = 0;
	int n_prev = 0;

	while (tries < 5) {
		int n = ev2citrusleaf_cluster_get_active_node_count(cluster);

		if (n > 0 && n == n_prev) {
			// Found a stable cluster of n nodes.
			return true;
		}

		sleep(1);
		tries++;
		n_prev = n;
	}

	return false;
}
```

You can use a series of timer events instead of a loop with sleep calls (for example, when a single-threaded application needs to wait for a stable cluster).

### Cleaning Up

The application must ensure that outstanding database transactions complete before it can destroy the cluster object and clean up the client.

```cpp
ev2citrusleaf_cluster_destroy(cluster);
ev2citrusleaf_cluster_shutdown(true);
```

If the application specifies the cluster management event base in the `create` call, it must ensure that this event base loop exited before it can call `ev2citrusleaf_cluster_destroy()`. Internally, `destroy` directly invokes the event loop to clear all outstanding cluster-management events. Also, the event base must not be freed until after the cluster destroy call.

```cpp
event_base_loopbreak(mgr_base);
// ... and perhaps join the cluster manager thread if it's a different thread.
ev2citrusleaf_cluster_destroy(cluster);
event_base_free(mgr_base);
ev2citrusleaf_cluster_shutdown(true);
```

### Accessing Multiple Clusters

To access more than one Aerospike database cluster from a single Aerospike libevent client, independently create, manage, and destroy each `ev2citrusleaf_cluster` object.

All database transaction functions in the API have a `cluster` object parameter. The application must specify the cluster for each transaction.
