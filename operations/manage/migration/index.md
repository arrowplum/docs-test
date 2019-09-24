---
title : Migrations
---

### Understanding Migrations

Aerospike balances data in the cluster by migrating it between cluster nodes. Migrations in an Aerospike cluster do not move single records. Data moves as a part of a partition and are monitored at a per-namespace basis. For each namespace, every record is mapped to one of 4096 partitions. The partitions are distributed throughout the cluster. When a cluster migrates data, it moves the partition atomically. Depending on your settings, more or fewer partitions may be migrated at the same time.

Migrations occur in the following scenarios:
1. Adding a node to the cluster.
2. Removing a node from the cluster.
3. Any network changes that leads to cluster size changes.

{{#info}}
Version 4.3.1 Enterprise Edition introduces the [`migrate-fill-delay`](/docs/reference/configuration/#migrate-fill-delay) parameter to control migrations. 
Refer to the [Delay Migrations page](/docs/operations/manage/migration/delay_migrations/index.html) for further details.
{{/info}}

### Monitoring Migrations

{{#info}}
Version 4.3 introduces the [`cluster-stable`](/docs/reference/info/#cluster-stable) info command. Running this command with 
the `ignore-migrations` flag set to `no` or `false` will return the [`cluster_key`](/docs/reference/metrics/?show-removed=1#cluster_key) 
only if migrations have completed.
{{/info}}

There are multiple ways to monitor migrations in the cluster:

1.**Using asadm**: The output of `asadm -e 'info namespace'` has details on the migrations status. The column for 'Migrates' shows the current status. If there are no ongoing migrations, the column shows (0,0). 

Refer to the [Aerospike Admin User Guide](/docs/tools/asadm/user_guide) for further details.

```asciidoc
Admin> info namespace
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace                   Node   Avail%   Evictions                 Master                Replica     Repl     Stop     Pending         Disk    Disk     HWM          Mem     Mem    HWM      Stop   
        .                      .        .           .   (Objects,Tombstones)   (Objects,Tombstones)   Factor   Writes    Migrates         Used   Used%   Disk%         Used   Used%   Mem%   Writes%   
        .                      .        .           .                      .                      .        .        .   (tx%,rx%)            .       .       .            .       .      .         .   
test        192.168.122.xx:3000   91         0.000     (976.732 K,0.000  )    (1.023 M,0.000  )      2        false    (0,0)       488.280 MB   6       50      253.694 MB   40      60     90        
test        192.168.122.yy:3000   91         0.000     (1.023 M,0.000  )      (976.732 K,0.000  )    2        false    (0,0)       488.280 MB   6       50      253.694 MB   40      60     90        
test                                         0.000     (2.000 M,0.000  )      (2.000 M,0.000  )                        (0,0)       976.561 MB                   507.389 MB
Number of rows: 3

```

2.**From the logs**: The log ticker (regular default 10 seconds interval) provides information about the migrations state for each namespace. Grepping the logs for 'migrations' will provide such line:

```asciidoc
cat /var/log/aerospike/aerospike.log | grep "migrations"

Apr 27 2016 18:19:59 GMT: INFO (partition): (partition.c::3195) {test} re-balanced, expected migrations - (0 tx, 2731 rx)

# The following line would show completed migrations:
Apr 29 2016 19:30:17 GMT: INFO (info): (thr_info.c::5063) {test} migrations - complete
```

Refer to [Server Log Messages](/docs/reference/serverlogmessages) for further details on log messages.

3.**From the statistics**: There is one service level statistic, [`migrate_partitions_remaining`](/docs/reference/metrics#migrate_partitions_remaining) which indicates whether there are any ongoing migrations on the current node. This statistic should be only consulted when [`migrate_allowed`](/docs/reference/metrics/#migrate_allowed) returns `true`. Other namespace level statistics are also available. Refer to this knowledge-base article on "[Monitoring migrations on a live cluster](https://discuss.aerospike.com/t/faq-monitoring-migrations-on-a-live-aerospike-cluster/3200)" for details on the metrics as per your server version.

## Tuning Migrations

Aerospike provides several configuration parameters to control the amount of migrations. The default values are intended, as much as possible, to have migrations traffic not interfer with the client application(s) transactions. Under specific situations, it may be necessary to alter those values in order to either speed up or slow down migrations traffic.

Parameter	| Description	| Default Value
--- | --- | ---
[migrate-threads](/docs/reference/configuration#migrate-threads) | Number of threads that perform migrations. | 1
[migrate-max-num-incoming](/docs/reference/configuration#migrate-max-num-incoming) | Maximum number of partitions a node can be receiving records from at any given time.  | 4
[migrate-sleep](/docs/reference/configuration#migrate-sleep) | Time that migrations sleep after migrating each record, in microseconds. | 1
[migrate-order](/docs/reference/configuration#migrate-order) | Number between 1 and 10 which determines the order namespaces are to be processed when migrating. Namespaces with lower migrate-order values process migrations first. Use this configuration to prioritize migrations by namespace. This configuration is useful for prioritizing migrations for in-memory namespaces, to minimize the risk of data loss if a node leaves the cluster during migrations. | 5
[channel-bulk-recv-threads](/docs/reference/configuration#channel-bulk-recv-threads) | Number of threads processing intra-cluster messages arriving through the bulk channel. This channel is used for record migrations during rebalance. | 4
[channel-bulk-fds](/docs/reference/configuration#channel-bulk-fds) | Number of bulk channel sockets to open to each neighbor node. Twice this number of sockets per neighbor will be opened since the neighbor nodes will open the same number of sockets back to this node. | 2

### Speeding up the migration rate

The following configuration parameters can be modified in order to speed up the migrations traffic.

{{#warn}}The cluster performance (throughput/latencies) should be monitored when any of those configuration parameter is modified to ensure the impact on the client application(s) traffic is acceptable. It is recommended to proceed by incremental steps and validate/monitor between each.{{/warn}}

- [`migrate-sleep`](/docs/reference/configuration/?show-removed=1#migrate-sleep) can be decreased to a minimum value of 0. 
- [`migrate-max-num-incoming`](/docs/reference/configuration/?show-removed=1#migrate-max-num-incoming) can be increased to allow more concurrent incoming partitions to a given node. In a cluster of N nodes, with MT migrate-threads, each node could theoretically receive a maximum of (N-1) x MT partitions concurrently. 
- [`migrate-threads`](/docs/reference/configuration/?show-removed=1#migrate-threads) can be increased. It is important to keep [`migrate-max-num-incoming`](/docs/reference/configuration/?show-removed=1#migrate-max-num-incoming) to a corresponding healthy value to avoid having some node potentially receiving too many partitions concurrently, especially when introducing a new empty node in a cluster.

The following 2 configuration parameters can also be tuned to accommodate even higher migrations traffic:

- [`channel-bulk-recv-threads`](/docs/reference/configuration/?show-removed=1#channel-bulk-recv-threads) can be increased from the default 4, to a higher number, for example, 8, or 16. Make sure the network bandwidth is also monitored when increasing those threads.
- [`channel-bulk-fds`](/docs/reference/configuration/?show-removed=1#channel-bulk-fds) can also be increased, but would require a restart as it is not dynamically configurable.


Here are the command to dynamically change those settings:

```
asadm -e "asinfo -v 'set-config:context=namespace;id=<namespace>;migrate-sleep=<sleep time in microseconds>'"
```

```
asadm -e "asinfo -v 'set-config:context=service;migrate-max-num-incoming=<max number of incoming partitions>'"
```

```
asadm -e "asinfo -v 'set-config:context=service;migrate-threads=<number of threads>'"
```

```
asadm -e "asinfo -v 'set-config:context=network;fabric.channel-bulk-recv-threads=<number of threads>'"
```

Finally, migrations can be prioritized across namespaces. The [`migrate-order`](/docs/reference/configuration/?show-removed=1#migrate-order) configuration parameter controls the priority between namespaces. The namespace with the lowest [`migrate-order`](/docs/reference/configuration/?show-removed=1#migrate-order) value is processed first. The namespace with the highest [`migrate-order`](/docs/reference/configuration/?show-removed=1#migrate-order) value is processed last.

Use the following command to modify [`migrate-order`](/docs/reference/configuration/?show-removed=1#migrate-order) for all nodes in the cluster:

```
asadm -e "asinfo -v 'set-config:context=namespace;id=<namespace>;migrate-order=<order value>'"
```

### Delaying fill migrations

As of version 4.3.1, the migrations can significantly be reduced to a minimum for usual maintenance operational procedure such as rolling restarts. For such operating procedures where nodes are brought down one at a time and brought back with data, the [`migrate-fill-delay`](/docs/reference/configuration/?show-removed=1#migrate-fill-delay) prevents unecessary migrations of unchanged data to other nodes in the cluster as those would 
be cancelled as the node that was taken down is brought back with data. Lead migrations (for the data that was changed while the node was out of the cluster) will still proceed. For example, if nodes will be taken down for no more than 1 hour at a time, the following command will prevent any fill migrations for up to 1 hour:

```
asadm -e "asinfo -v 'set-config:context=service;migrate-fill-delay=3600'"
```

Refer to the [`migrate-fill-delay`](/docs/reference/configuration/?show-removed=1#migrate-fill-delay) configuration parameter for further details.

### Slowing down the migration rate

If migrations do seem to impact the performance of the cluster, and for cases where the above mentioned [`migrate-fill-delay`](/docs/reference/configuration/?show-removed=1#migrate-fill-delay) cannot be leveraged, the migration rate can be slowed down. Assuming default configurations were kept, the best way to slow the migration rate is to increase the [`migrate-sleep`](/docs/reference/configuration/?show-removed=1#migrate-sleep) configuration. [`migrate-sleep`](/docs/reference/configuration/?show-removed=1#migrate-sleep) is the number of microseconds that migrations sleep between each record transmitted out. The [`migrate-max-num-incoming`](/docs/reference/configuration/?show-removed=1#migrate-max-num-incoming) can then be decreased (default 4) to reduce the number of partitions a node can receive concurrently. The [`migrate-threads`](/docs/reference/configuration/?show-removed=1#migrate-threads) should in such scenario be kept at its default (1) to ensure at most 1 partition migrated at a time on any node. Setting [`migrate-threads`](/docs/reference/configuration/?show-removed=1#migrate-threads) to 0 will pause migrations, but only after the  partitions which were migrating, do complete.
