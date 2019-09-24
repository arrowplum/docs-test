---
title: Storage Format Upgrade in 4.2 Release
description: Steps for upgrading to 4.2 and the new storage format.
styles:
  - /assets/styles/ui/steps.css
---

In server version 4.2, Aerospike has changed the internal storage format. This change requires all storage devices zeroized or files to be deleted as part of the upgrade to version 4.2. The internal storage format introduced in 4.2 improves on efficiency and speed. It also allows for configuring the [`write-block-size`](/docs/reference/configuration/#write-block-size) up to 8MiB.

{{#info}}
Versions 4.2 and pre-4.2 are **cluster-compatible**, so a rolling restart upgrade with mixed versions is supported. Please keep a copy of your original aerospike.conf configuration file, in case of an unlikely event requiring a downgrade.
{{/info}}


{{#warn}}
As persisted data has to be erased before upgrading a node, it is essential to wait for migrations to complete prior to proceeding to the next node, allowing the data to be repopulated from other nodes in the cluster (assuming replication factor 2 or higher for the namespace). The Aerospike asd daemon will not start if the storage has not been deleted.
{{/warn}}

## Configuration changes
The following 2 configuration parameters are affected by version 4.2:

1. [`partition-tree-locks`](https://www.aerospike.com/docs/reference/configuration/#partition-tree-locks) has been deprecated and should be removed from the configuration file if previously specified. (Now hardcoded to 256).

2. [`partition-tree-sprigs`](https://www.aerospike.com/docs/reference/configuration/#partition-tree-sprigs) has its minimum allowed value raised to 256. Changing this configuration parameter requires a cold restart but for this upgrade the data stored has to be erased, it is a good opportunity to consider increasing the number of sprigs.


## Upgrade steps

{{#info}}Be familiar with the [general guide lines for upgrading a cluster](/docs/operations/upgrade/aerospike/index.html) for the common steps involved in an Aerospike cluster upgrade.
{{/info}}

{{#warn}}Persisted data must be deleted prior to starting a node with the 4.2 version. It is recommended to backup the data or have a redundant cluster prior to proceeding with the upgrade.{{/warn}}

{{#note}}Namespaces with replication factor of 1 would require their data restored from a backup or through XDR or other clients.{{/note}}

For each node in the cluster:

1. Update the configuration file to reflect the changes mentioned above regarding [`partition-tree-locks`](https://www.aerospike.com/docs/reference/configuration/#partition-tree-locks) and [`partition-tree-sprigs`](https://www.aerospike.com/docs/reference/configuration/#partition-tree-sprigs).
2. Stop the Aerospike daemon.
3. Delete the stored data for storage-engine device configured namespaces. Simply delete the file if using a file. For raw devices, use blkdiscard or dd. Refer to the [Zeroize Multiple SSD Simultaneously](https://discuss.aerospike.com/t/zeroize-multiple-ssds-simultaneously/716) knowledge base article. 

{{#note}}To protect against the unlikely event of an irrecoverable node crash while a node has been taken out to be upgraded, wait for migrations to complete prior to deleting the stored data to ensure 2 copies of the data are always available.{{/note}}

4. Start the Aerospike daemon.
5. Wait for migrations to complete after the node joins the cluster to allow all the data to be repopulated from other nodes (assuming replication factor 2 or more). Refer to the [Monitoring Migrations on a Live Cluster](https://discuss.aerospike.com/t/faq-monitoring-migrations-on-a-live-aerospike-cluster/3200) knowledge base article. 
6. Proceed with the next node.

## Important note

It is recommended to check, prior to the upgrade, whether there are potential write transactions failing due to the record size being larger than the configured [write-block-size](/docs/reference/configuration/#write-block-size). Those can be tracked through the [fail_record_too_big](/docs/reference/metrics/#fail_record_too_big) metric. As the record overhead is reduced in version 4.2, it is possible that records previously failing to be written get a reduction of overhead permitting them to be successfully written. This would cause potential issues during migrations and replica writes to nodes not already upgraded to fail. Proceeding normally with the rolling upgrade, one node at a time should complete successfully even if migrations do temporarily get stuck (trying to transmit a record resulting with a larger overhead on a node not upgraded yet). 

{{#info}}
If nodes are actually replaced with new ones, it may be 
necessary to preserve their [`node-id`](/docs/reference/configuration/#node-id) to make sure a new node takes over the same partitions as a node that has been removed and avoid migrations to potentially get stuck if such records with problematic size exist. 
{{/info}}

If using XDR, it is recommended to first upgrade a destination DC (for active / passive topologies) to avoid failing writing records at a destination after a source cluster is upgraded to 4.2.

In general, applications should avoid writing records larger (or very close to) the write-block-size.

{{#info}}
Refer to the [4.2 FAQ](/docs/operations/upgrade/storage_to_4_2/faq/index.html) for other info regarding this procedure.
{{/info}}
