---
title: Frequently asked questions regarding the 4.2 storage format change
description: Frequently asked question regarding the 4.2 storage format change.
---

### **Downgrade Options**

#### Is a downgrade possible after the upgrade to the new storage format is completed?

Starting Aerospike 4.2, [write-block-size](/docs/reference/configuration#write-block-size) can be configured to a maximum of 8 MiB. Refer to the [Storage format upgrade in 4.2](/docs/operations/upgrade/storage_to_4_2/index.html) page details on the upgrade procedure.

A downgrade is possible with the following steps, depending on the configuration.

##### **If any record on 4.2 has a size greater than 1 MiB**

Delete the records that have a size (overhead included) greater than 1 MiB using a client application and proceed to the section below when all records are smaller than 1 MiB in size. If this is not a possible or practicle solution, follow those steps:
1. Stop all writes to the cluster and take a backup of the cluster.
2. Stop Aerospike daemon across all nodes in the cluster nodes and clean up the data on the device or file. 
  * Simply delete the file if using a file. 
  * For raw devices, use blkdiscard or dd. Refer to the [Zeroize Multiple SSD Simultaneously](https://discuss.aerospike.com/t/zeroize-multiple-ssds-simultaneously/716) knowledge base article for details on how to use those commands.
3. Downgrade the binary to the previous desired version.
  * Depending on the version, some configuration may have to be changed. 
  * Downgrading to version 3.13 or before requires more steps, refer to the [3.13 FAQ](https://www.aerospike.com/docs/operations/upgrade/cluster_to_3_13/faq/index.html) for details.
4. Remove records with size greater than 1 MiB from the backup files. Enterprise Licensees can contact Aerospike Support for recommendations on doing this.
5. Start the Aerospike server across all nodes in the cluster nodes and restore the data from backup. 

##### **If all records on 4.2 have a size smaller than 1 MiB**

{{#note}}
The following assumes namespaces of replication factor 2 or more with persisted data.
{{/note}}

1. Stop the Aerospike server on one of the nodes in the cluster.
2. As the persisted data would have to be cleared, it is suggested to wait for migrations to complete before proceeding to the next step to avoid loss of data in the unlikely event of an irrecoverable hardware failure on a different node.
3. Clean up the data on the device or file for the node.
  * Simply delete the file if using a file. 
  * For raw devices, use blkdiscard or dd. Refer to the [Zeroize Multiple SSD Simultaneously](https://discuss.aerospike.com/t/zeroize-multiple-ssds-simultaneously/716) knowledge base article for details on how to use those commands.
4. Downgrade the binary to the previous desired version 
  * Depending on the version, some configuration may have to be changed. 
  * Downgrading to version 3.13 or before requires more steps, refer to the [3.13 FAQ](https://www.aerospike.com/docs/operations/upgrade/cluster_to_3_13/faq/index.html) for details.
5. Start the Aerospike server on the node and confirm it has joined the cluster (check the [cluster_size](/docs/reference/metrics#cluster_size) metric across all nodes in the cluster).
6. Wait for migrations to complete after the node joins the cluster to allow all the data to be repopulated from other nodes. Refer to the [Monitoring Migrations on a Live Cluster](https://discuss.aerospike.com/t/faq-monitoring-migrations-on-a-live-aerospike-cluster/3200) knowledge base article.
7. Proceed with the next node.

{{#note}}
There is a known bug in versions 4.1 and 4.2.0.2 where records with sizes close to 1 MiB can cause a segmentation fault. This  was addressed in server version 4.2.0.3.
{{/note}}
