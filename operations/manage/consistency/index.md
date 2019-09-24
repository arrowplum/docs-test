---
title: Consistency Management
description: Managing Strong Consistency in the Aerospike Database
---

This section covers how to add and remove nodes in strong consistency namespaces,
as well as how to detect and repair unavailability.

# Adding Additional Nodes

This section will guide you through the process of adding a node to an existing namespace configured with "strong-consistency".

## Install & Configure the Additional Nodes

Install and configure Aerospike on the new nodes as you had done in ["Install & Configure for Strong Consistency
"](/docs/operations/configure/consistency/index.html#).

When the nodes have joined the cluster we will see the service's `cluster_size` being greater than `ns_cluster_size` - 
also the `roster:` command will show the newly observed nodes in its `observed_nodes`.

```
Admin> show stat -flip like cluster_size
~~~~~~~~~~~~~~~~~~~~~~~~Service Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            cluster_size
node1.aerospike.com:3000                             6
node2.aerospike.com:3000                             6
node4.aerospike.com:3000                             6
node5.aerospike.com:3000                             6
node6.aerospike.com:3000                             6
node7.aerospike.com:3000                             6
Number of rows: 6

~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            ns_cluster_size
node1.aerospike.com:3000                             5
node2.aerospike.com:3000                             5
node4.aerospike.com:3000                             5
node5.aerospike.com:3000                             5
node6.aerospike.com:3000                             5
node7.aerospike.com:3000                             5
Number of rows: 6

Admin> asinfo -v "roster:namespace=test" with BB9080016AE4202
node8.aerospike.com (192.168.10.8) returned:
        roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202
observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
```

### Adding the new nodes to the Roster

**Copy the observed_node list from the `roster:`** command and issue a `roster-set:` 
using this list as the value to the `nodes` parameter:

```
Admin> asinfo -v "roster-set:namespace=test;nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202"
node2.aerospike.com:3000 (192.168.10.2) returned:
ok

node7.aerospike.com:3000 (192.168.10.7) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ok

node5.aerospike.com:3000 (192.168.10.5) returned:
ok

node4.aerospike.com:3000 (192.168.10.4) returned:
ok

node1.aerospike.com:3000 (192.168.10.1) returned:
ok
```

**Activate the new roster** with a `recluster:`:

```
Admin> asinfo -v "recluster:"
node2.aerospike.com:3000 (192.168.10.2) returned:
ignored-by-non-principle

node7.aerospike.com:3000 (192.168.10.7) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ignored-by-non-principle

node5.aerospike.com:3000 (192.168.10.5) returned:
ignored-by-non-principle

node4.aerospike.com:3000 (192.168.10.4) returned:
ignored-by-non-principle

node1.aerospike.com:3000 (192.168.10.1) returned:
ignored-by-non-principle
```

**Run the `roster:` command** and confirm that the roster has been updated on all nodes to the 
roster you had set. Also make certain the service's `cluster_size` matches the namespace's `ns_cluster_size`.

Congratulations, you have successfully added nodes to your cluster.

## Removing Nodes

This section will guide you through the process of removing a node from an existing namespace configured with strong-consistency.

The general process is to remove a node from a cluster first. This will begin the data migration process, and after
the data is safely replicated elsewhere, you can remove the node from the namespaces's roster.

*** Important ***: When removing nodes, take care to not remove replication-factor or more nodes at a time. 
Removing too many nodes simultaneously will result `dead_partitions`, which is your signal of potential data loss.
If you observe dead partitions, you may wish to re-add the nodes and wait for the cluster to synchronize before proceeding
again.

{{#note}}
**Note** - Strong consistency implies the guarantee that with replication factor of N, N copies of data will be written to the cluster.  A fully formed cluster must contain X nodes where X >= N to satisfy this.  Creating a cluster where X = N will mean that all partitions become unavailable during a single node shutdown.
{{/note}}

{{#note}}
**Note** - Namespaces with replication-factor set to 1 will have some partitions unavailable whenever **any** node leaves the cluster, making it impractical to perform a rolling restart/upgrade.
{{/note}}

### Removing the Nodes from the Cluster

**Execute a `roster:` command** to make sure all roster nodes are present in the cluster

```
asinfo -v "roster:namespace=test" -l
```

```
Admin> asinfo -v "roster:namespace=test" -l
node1.aerospike.com:3000 (192.168.10.1) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node5.aerospike.com:3000 (192.168.10.5) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node4.aerospike.com:3000 (192.168.10.4) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node3.aerospike.com:3000 (192.168.10.3) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node6.aerospike.com:3000 (192.168.10.6) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node7.aerospike.com:3000 (192.168.10.7) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
```



**Shutdown the nodes to be removed** - remember, fewer than replication-factor.

```
systemctl stop aerospike
```

**Execute a `show stat unavailable` command** to make sure there are no `unavailable_partitions`. 

If an unrelated failure happened during this process, 
restart the downed nodes, wait for migrations to complete, and restart this procedure.

```
Admin> show stat like unavailable for test -flip
~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            unavailable_partitions
node1.aerospike.com:3000                             N/E
node2.aerospike.com:3000                             5
node4.aerospike.com:3000                             5
node5.aerospike.com:3000                             5
node6.aerospike.com:3000                             5
node7.aerospike.com:3000                             5
Number of rows: 6
```

**Wait for migrations to complete.** Execute a stat command on `partitions_remain` until the remaining partitions become zero on all nodes.

```
Admin> show stat service like partitions_remain -flip
~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            migrate_partitions_remaining
node1.aerospike.com:3000                             N/E
node2.aerospike.com:3000                             0
node4.aerospike.com:3000                             0
node5.aerospike.com:3000                             0
node6.aerospike.com:3000                             0
node7.aerospike.com:3000                             0
Number of rows: 6
```

We will now remove the node from the roster, which involves setting the roster to all the 
nodes except the node we are removing.

**Execute a `roster:` command** to retrieve the current roster.

Notice that the observed nodes is one smaller than the `roster` and `pending_roster`. Externally, validate
that the missing node's ID is the one you have removed from the cluster.

```
Admin> asinfo -v "roster:namespace=test" -l
node1.aerospike.com:3000 (192.168.10.1) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node5.aerospike.com:3000 (192.168.10.5) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node4.aerospike.com:3000 (192.168.10.4) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node6.aerospike.com:3000 (192.168.10.6) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node2.aerospike.com:3000 (192.168.10.2) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202

node7.aerospike.com:3000 (192.168.10.7) returned:
roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
pending_roster=BB9070016AE4202,BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
observed_nodes=BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202
```

**Copy the `observed_node` list and issue a `roster-set:`** using this list as the value to the nodes parameter.

```
Admin> asinfo -v "roster-set:namespace=test;nodes=BB9060016AE4202,BB9050016AE4202,BB9040016AE4202,BB9020016AE4202,BB9010016AE4202"
```

**Issue a `recluster:`** to apply the change. Note that `asinfo` still knows about the node you have removed,
so there is an expected error connecting to that node.

```
Admin> asinfo -v "recluster:"
node7.aerospike.com:3000 (192.168.10.7) returned:
Error: Could not connect to node 192.168.10.7

node2.aerospike.com:3000 (192.168.10.2) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ignored-by-non-principle

node5.aerospike.com:3000 (192.168.10.5) returned:
ignored-by-non-principle

node4.aerospike.com:3000 (192.168.10.4) returned:
ignored-by-non-principle

node1.aerospike.com:3000 (192.168.10.1) returned:
ignored-by-non-principle
```


# Server Shutdown

The safe way to operationally shutdown a server is to execute standard process stop commands. For example:
```
systemctl stop aerospike
```

But any operational procedure that executes a `SIGTERM` is safe. After startup, a `SIGTERM` will flush data to disk and properly signal other servers.

A special case is restarting during startup. 
Please avoid executing a process shutdown during startup - and reference the section on Dead Partitions to learn more. 

To check if startup is complete, run
```
asinfo -h [ip of host] -v 'status'
```
periodically until it returns "ok".

In the future, we plan to enable safe shutdowns during startup, which will render this check unnecessary.

# Validating Partition Availability

In cases such as network partitions and server hardware failure, partitions can become unavailable or dead. 
A dead partition is a special kind of unavailable: for full explanation see the following section.

After making a change, you will often expect all partitions to be available. This is a good command to execute 
after any operational change where you expect all data to be available.

The following command shows - for each node in the cluster - which nodes believe there are dead or unavailable 
partitions. Please note that each node will report the global number of dead or unavailable partitions. 
For example, if the entire cluster has determined that 100 partitions are unavailable, all of the current 
nodes will report that there are 100 partitions are unavailable.

Note: please do NOT use the asadm 'show pmap' command. This command has not been updated to 4.0 partition states.
 
```
show stat namespace for [namespace name] like 'unavailable|dead' -flip
```

```
Admin> show stat namespace for test like 'unavailable|dead' -flip
~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            dead_partitions  unavailable_partitions
node1.aerospike.com:30000                               0                       0
node2.aerospike.com:30000                               0                       0
node4.aerospike.com:30000                               0                       0
node5.aerospike.com:30000                               0                       0
node6.aerospike.com:30000                               0                       0
Number of rows: 5
```

In the output of this command, the columns "unavailable" and "dead" should be 0 for each node. 

If you have unavailable partitions, then there may be missing data. There are nodes which are 
expected in the roster but are not currently in the cluster. 
Please validate that your rosters are as expected. If roster nodes are missing from the cluster, 
please take the initial step of restoring the missing nodes by changing the roster, or 
fixing hardware or network issues.

If you have dead partitions, it means that all the roster nodes are in the cluster, 
but partitions are still unavailable due to storage failure or potential loss due to buffered writes. 
Please see the following discussion of dead partitions to determine the correct next operational step.

# Dead Partitions

Certain failures of hardware, combined with certain Aerospike configurations, may result in cases where data has been lost. 

The cases that may result in data loss are:
1. Drive loss, such as user actions to erase drives ( clearing or wiping ), or catastrophic hardware failure
1. If not commit-to-device, untrusted or unclean shutdown ( crash, SIGKILL, SIGSEGV, and others )
1. Clean shutdown during a Fast Restart ( potentially removed in future versions, as data loss is not possible )

In these cases, you may or may not have lost data --- depending on the number of nodes simultaneously 
affected, the replication factor, and whether data migration has completed for the partitions in question. 

Aerospike detects whether you have multiple failures in these cases, and detects potential true data loss. 
Affected partitions are marked "dead", and require operator intervention to continue. To continue to allow
reads would violate Strong Consistency.

The effect of unclean shutdown can be limited by setting the `commit-to-device` namespace option. 
With the `commit-to-device` option, simultaneous crashes are known not to lose data, thus never 
generate dead partitions. However, enabling `commit-to-device` generates a flush on every write, and 
thus comes with performance penalties that should be measured - but may be minimal for low write throughput 
cases, or on high performance storage such as some direct attach NVMe drives or some high performance 
enterprise SANs.

If you do not have `commit-to-device` enabled, you will often see server crashes generating 
"dead" partitions. User-generated restarts (upgrades) will not generate this effect, as a restart 
will flush to disk on shutdown even without commit-to-device. 

Dead partitions are detected though the commands in the above section 
regarding Validating Partition Availability above.

In the case of detected potential data loss, the cluster maintains unavailable dead partitions 
in order to allow you to take corrective action. You may decide that no data has been lost 
(such as shutdown during Fast Restart, running a test cluster, determination that availability 
is more important than correctness in this case,you may decide that availability in the case of 
this potential data loss is preferable to your business, or you may decide to restore data from an 
external source ( if available ). 
If you do have an external trusted source, you might consider disabling applications, 
reviving the potentially flawed namespace, restoring from the external trusted source, 
then enabling applications. With Aerospike's feature that alerts you of these conditions, consistent 
reads are guaranteed allowing time for correct operator intervention.

## Reviving Dead Partitions:

This process should be followed in the operational cases where you wish to use your namespace
in the face of missing data. For example, you may have entered a maintenance state where you
have disabled application use, and are preparing to reapply data from a reliable message queue
or other source.

**Notice that you have dead partitions**
```
show stat namespace for test like dead -flip
```

```
Admin> show stat namespace for test like dead -flip
~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            dead_partitions 
node1.aerospike.com:3000                              264
node2.aerospike.com:3000                              264
node4.aerospike.com:3000                              264
node5.aerospike.com:3000                              264
node6.aerospike.com:3000                              264
Number of rows: 5
```

**Execute `revive:`** to acknowledge the potential data loss on each server.
```
asinfo -v "revive:namespace=[test]"
```

```
Admin> asinfo -v "revive:namespace=test"
node2.aerospike.com:3000 (192.168.10.2) returned:
ok

node1.aerospike.com:3000 (192.168.10.1) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ok

node5.aerospike.com:3000 (192.168.10.5) returned:
ok

node4.aerospike.com:3000 (192.168.10.4) returned:
ok
```

**Execute "recluster:"** to enliven the dead partitions. 
```
asinfo -v "recluster:"
```

```
Admin> asinfo -v "recluster:"
node2.aerospike.com:3000 (192.168.10.2) returned:
ignored-by-non-principal

node1.aerospike.com:3000 (192.168.10.1) returned:
ok

node6.aerospike.com:3000 (192.168.10.6) returned:
ignored-by-non-principal

node5.aerospike.com:3000 (192.168.10.5) returned:
ignored-by-non-principal

node4.aerospike.com:3000 (192.168.10.4) returned:
ignored-by-non-principal
```

**Verify that there are no longer any dead partitions** using the dead_partitions stat.
```
show stat like dead_partitions -flip
```


```
Admin> show stat namespace for test like dead -flip
~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Statistics~~~~~~~~~~~~~~~~~~~~~
                          NODE            dead_partitions 
node1.aerospike.com:3000                                0 
node2.aerospike.com:3000                                0 
node4.aerospike.com:3000                                0 
node5.aerospike.com:3000                                0 
node6.aerospike.com:3000                                0 
Number of rows: 5
```
