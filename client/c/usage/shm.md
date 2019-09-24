---
title: Shared Memory
description: Enable shared memory for client cluster tending.
---

Every client instance runs an extra cluster tend thread that periodically polls server nodes for cluster status. In a single-process/multi-threaded environment, only one client instance is required. This instance is shared across all threads. However, in a multi-process/single-thread environment, multiple client instances are required, which means that multiple cluster tend threads are simultaneously polling server nodes for cluster status. Shared memory resolves this inefficiency.

Multi-process/single-threaded environments are common for languages like PHP and Python. The Aerospike PHP and Python clients are implemented using calls to the Aerospike C client, gaining access to the C client's shared memory capabilities.

When shared memory is enabled, cluster state (including nodes and data partition maps) is stored in a shared memory segment. Only one cluster tend owner process polls server nodes for cluster status and writes to this shared memory segment. All other client processes read cluster status from this shared memory segment, and do not poll the server nodes.

One shared memory segment stores state for a single cluster.  Multiple clusters require multiple shared memory segments.  Each shared memory segment is identified by the user configurable `shm_key`. 

### Configuration

Configure shared memory using these fields in `as_config`:

```cpp
bool use_shm;
int shm_key; 
uint32_t shm_max_nodes;
uint32_t shm_takeover_threshold_sec;
```

- `use shm` &mdash; Set to true to use shared memory for cluster tending. Shared memory is useful when operating in single-thread mode with multiple client processes. This model is used by wrapper languages such as PHP and Python. When enabled, data partition maps are maintained by only one process and all other processes use these shared memory maps. Default: false

- `shm_key` &mdash; Identifier for the shared memory segment associated with the target Aerospike cluster.  Each shared memory segment contains state for one Aerospike cluster.  If there are multiple Aerospike clusters, a different shm_key must be defined for each cluster. Default: 0xA7000000
 
- `shm_max_nodes` &mdash; Set the maximum number of server nodes allowed using shared memory and the size of the fixed shared memory segment. Default: 16

{{#note}}
Ensure that you leave a cushion between actual server node count and `shm_max_nodes` so that you can add new nodes without rebooting the client.
{{/note}}

- `shm_max_namespaces` &mdash; Set the maximum number of namespaces, which sizes the fixed shared memory segment. Default: 8

{{#note}}
Ensure that you leave a cushion between actual namespace count and `shm_max_namespaces` so that you can add new namespaces without rebooting the client.
{{/note}}

- `shm_takeover_threshold_sec` &mdash; Take over shared memory cluster tending if the cluster is not tended by this threshold within _n_ seconds. Default: 30

### Enable Shared Memory

To enable shared memory before calling `aerospike_connect()`:

```cpp
as_config config;
as_config_init(&config);
as_config_add_host(&config, host, port);
config.use_shm = true;
config.shm_key = 0xA7000000;
config.shm_max_nodes = 16;
config.shm_max_namespaces = 8;
config.shm_takeover_threshold_sec = 30;

aerospike client;
as_error err;
aerospike_init(&client, &config);
as_status status = aerospike_connect(&client, &err);
```

### Operational Notes

- The shared memory segment is automatically created by the client if it does not already exist on that machine.

- The first client process owns cluster tending by acquiring a shared memory lock.

- Ensuing client processes read cluster status from the shared memory segment, but do not cluster tend nor write to shared memory.

- Applications must call `aerospike_close()` before exiting to release the shared memory lock. Another client process automatically acquires the lock and becomes the new cluster tender.

- If the application exits without releasing the lock, another client process will take over cluster tending after the takeover threshold is reached (`shm_takeover_threshold_sec`, default = 30 seconds). Cluster status can still be read during this lag, but cluster tending (polling servers for cluster status) does not commence until another client process takes over.

- You can view shared memory segments with this command:

  ```bash
  ipcs -m

  ------ Shared Memory Segments --------
  key        shmid      owner      perms      bytes      nattch     status
  0xA7000000 622592     root       666        263224     0 
  ```

- To change shared memory configuration variables (for example, to increase max nodes), a new shared memory segment must be created. There are two ways to upgrade your application:

    - **One Shot Upgrade**
	1. Close all applications.
	2. Delete the shared memory segment (if it still exists):

        ```bash
        ipcrm -M 0xA7000000  
        ```

	3. Restart your applications.

    - **Rolling Upgrade**
	1. Set a different shared memory key in your new application:
        ```cpp
        config.shm_key = 0xA7000001;
        ```
	New application instances use the new shared memory segment, automatically created by the first new application instance. Old applications instances continue to use the old shared memory segment.
	2. Gradually stop all old application instances and replace them with new application instances.
	3. Delete the old shared memory segment (if it still exists):
        ```bash
        ipcrm -M 0xA7000000  
        ```
