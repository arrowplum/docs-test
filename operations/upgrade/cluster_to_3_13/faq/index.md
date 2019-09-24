---
title: Frequently asked questions regarding the 3.13 upgrade
description: Frequently asked question regarding the 3.13 cluster protocol change.
---

#### 1. Some hosts in the cluster successfully switched to paxos v5, but some did not.
* Re-apply the dynamic switch if still within the extended heartbeat timeout and if the cluster has not split.
* If the cluster has split, first retain the paxos v5 cluster, then restore any heartbeat timeout extension to normal, and assimilate the paxos v3 nodes, one-by-one in any of the following way:
	* Dynamically split the paxos v3 nodes each as a single-node, by assigning unique cluster-name to each v3 node. Run the command on the latest tools package. 
			* asinfo -v 'set-config:context=service;cluster-name="cluster1"' 
	* Re-apply dynamic switch on each v3 node. 
			* asinfo -v 'set-config:context=service;paxos-protocol=v5'
	* Remove or switch back the cluster-name to the original value. If there was no cluster name earlier, run the command below else replace null with the name.
			* asinfo -v 'set-config:context=service;cluster-name=null'
	* Take all v3 nodes down. Change each paxos v3 node's static configuration to paxos v5, restart each node to join the cluster. 
		* **NOTE** - each rejoined node's partition gets a new version, so there will be more duplicate resolving and data migrations.
* **NOTE** - dynamic switch is not allowed to execute when the cluster is not well-formed or migrating.

#### 2. After switching a cluster over to paxos v5 and running for a while, how to add an old paxos v3 node?
* Any of the above are options. But if the node to be added has been out of the cluster for a long time, it is recommended to remove all data (wipe the disks - including System Metadata(SMD)) and have the node rejoin as an empty node, with the configuration set to the new protocol.

#### 3. Is there an easier way to perform the switch over to the new protocol if the cluster can be taken fully down?
* There is no way to proceed with the upgrade while the cluster is down. The SAME dynamic switch is needed and requires the cluster to be up.
* However, in the unlikely case that only part of the cluster is switched over, stopping all write traffic during the procedure would prevent any write 
loss upon recovering from such state.

#### 4. What happens if a node is restarted with its configuration file not updated after the protocol has been switched?
* The restarted node will be in paxos v3, and will not join the cluster by default. One of the options provided in point 1 above would have to be applied.

#### 5. Does paxos version change affect remote clusters?
* No. Each data-center can be switched over independently.

#### 6. What happens to an upgrade that did not "jump" onto 3.13 and directly went to 3.14 or later?
* Upon start up the code will check the secondary index(sindex) System Metadata(SMD) file name.
   * Pre-upgrade name  : sindex_module.smd
   * Post-upgrade name : sindex.smd
* The following error would be logged: "FAILED ASSERTION (smd): (system_metadata.c:1305) Aerospike server was not properly switched to paxos-protocol v5 - see Aerospike documentation https://www.aerospike.com/docs/operations/upgrade/cluster_to_3_13"
* If the upgrade fails and all parameters are updated, it may be an issue with the System Metadata(SMD) file.
   * If the original System Metadata(SMD) file sindex_module.smd is still in place check the file permissions and any accompanying warnings in the logs.  If this is determined to be the case, simply delete the original System Metadata(SMD) file sindex_module.smd and restart (multiple node required in order for the sindex smd file to be re-synced from other nodes in the cluster).
* Future versions will note that a switch was not done, and will not start.

#### 7. ** Downgrade Options **
* Is a downgrade possible if the paxos protocol has not been switched over to v5?
	* A downgrade is possible in such case, but the XDR digest log must be removed before restarting the downgraded node.
* Is a downgrade possible after the paxos protocol has been switched over to v5?
	* A downgrade would require a complete cluster shutdown.
	* paxos-protocol needs to be set to v2 or v3 in aerospike.conf.
	* The XDR digest log must be removed (as required for general downgrade from 3.13).
	* The sindex System Metadata(SMD) format is not backward compatible. As a result, any sindex will need to be manually recreated.
		
