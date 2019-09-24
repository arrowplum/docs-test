---
title: Clustering Protocol Upgrade for rack-aware
description: Operational procedure for clustering protocol upgrade in 3.13.
---

## Procedure for switch over to the new cluster protocol for **rack-aware enabled** systems

* This first step of the procedure are the same as the [general steps](/docs/operations/upgrade/cluster_to_3_13).

* Once the dynamic switch to the new clustering protocol is done, update the configuration file as follows for each node and proceed with a rolling restart:

	* Remove the entire cluster stanza from the config file.
	* For each namespace, add the [`rack-id`](/docs/reference/configuration#rack-id) configuration entry matching the `self-group-id` previously defined in "cluster" stanza.
	* If a `self-node-id` was specified, replace it with the [`node-id`](/docs/reference/configuration#node-id) parameter at the service level **when upgrading straight to version 3.16.0.1** post cluster protocol upgrade.

* Upgrading the clustering protocol on a **rack-aware enabled cluster** without explicitly preserving node IDs (using the [`node-id`](/docs/reference/configuration#node-id) configuration parameter introduced in version 3.16.0.1) would cause the node ID of each node to change upon their following restart. This can potentially cause a higher volume of migrations as partitions ownership would change upon the first restart of each node following the clustering protocol upgrade.

{{#warn}}
The rack-aware feature is an Aerospike Enterprise Edition Server only feature as of version 4.0.
{{/warn}}

{{#warn}}
Having the node ID of each node change upon their following restart also means that the number of records will temporarily increase while migrations are ongoing. This can be tracked under the [non_replica_objects](/docs/reference/metrics/#non_replica_objects) and 
[non_replica_tombstones](https://www.aerospike.com/docs/reference/metrics/#non_replica_tombstones) statistics. The number of such 
transient objects can be large and should be taken in consideration, especially if a rolling restart is triggered where migrations do 
not complete between each node's restart.
{{/warn}}

{{#note}}
The `self-node-id` configuration is deprecated post cluster protocol switch. If necessary, refer to [`node-id`](/docs/reference/configuration#node-id) for versions 3.16.0.1 
and above to specify the node ID or [`node-id-interface`](/docs/reference/configuration#node-id-interface) for earlier version to specify the interface to be used for the node id generation.
{{/note}}

{{#warn}}
The configuration file options [`node-id`](/docs/reference/configuration#node-id) and [`node-id-interface`](/docs/reference/configuration#node-id-interface) are mutually exclusive. 
{{/warn}}
