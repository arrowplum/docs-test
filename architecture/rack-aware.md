---
title: Rack Awareness
description: Rack awareness allows you to specify that master data and replicated data are not stored on servers in the same hardware failure group.
assets: /docs/architecture/assets
---

The Aerospike **Rack Awareness** feature allows you to store different replicas of records on different hardware failure groups (for example, if [`replication-factor`](https://www.aerospike.com/docs/reference/configuration/?show-removed=1#replication-factor) is 2, the master copy of a partition and its replica will be stored on different hardware failure groups). These groups are defined by their [`rack-id`](https://www.aerospike.com/docs/reference/configuration/index.html#rack-id)

{{#note}}
Rack awareness is only available in Aerospike Server version 3.x and above. If you are using a previous version, you must also upgrade your Aerospike software.
The rack-aware feature is an Aerospike Enterprise Edition Server only feature as of version 4.0.
{{/note}}

By default, replica partitions are evenly distributed across all cluster nodes. Each node (of a cluster containing _n_ nodes)  will approximately own 1/nth of the data partitions as master. The replicas for these partitions are evenly distributed between the other _n_-1 nodes (in total, each node owns approximately _rf_/_n_ where _rf_ is the [`replication-factor`](https://www.aerospike.com/docs/reference/configuration/?show-removed=1#replication-factor)). When a physical rack fails (or Availability Zone in case of a cloud deployment) those nodes can contain both master and replica partitions for the same partition, resulting in possible data unavailability.

To help avoid such data unavailability, the rack-aware feature can be configured to use multiple racks (defined by [`rack-id`](https://www.aerospike.com/docs/reference/configuration/index.html#rack-id)).

When the [`replication-factor`](https://www.aerospike.com/docs/reference/configuration/?show-removed=1#replication-factor) is less than or equal to the number of racks, _replication factor_ number of racks are ensured to have a copy of the partition, for each partition.

If the replication factor is greater than the number of racks, all racks will have a copy of the partition's data, and some racks will have additional copies of partitions, for each partition. 

If a single node goes down, the cluster is temporarily unbalanced until the node is restored. This imbalance does not cause service interruptions. The cluster continues uninterrupted. Once the node restarts, the cluster automatically rebalances.

{{#note}}
A couple of examples to illustrate the way this feature currently operates:<br>
- When configuring 3 racks along with replication factor of 3, each rack will receive one replica for a given partition.<br>
- When losing a rack under such configuration, assuming the [paxos-single-replica-limit](/docs/reference/configuration/#paxos-single-replica-limit)
 has not been breached, each partition will still have 3 replicas, the first replica (master) will be on one rack, the second one on a different one, and the third one would be on either one of the 2 racks, depending on the succession list order.
 {{/note}}

For more details on how to configure rack-aware, please see [Rack-aware](/docs/operations/configure/network/rack-aware) section under Operations.
