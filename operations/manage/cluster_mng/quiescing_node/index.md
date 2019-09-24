---
title : Quiescing a node
---

### Context

When the [`quiesce`](/docs/reference/info/#quiesce) command (Enterprise Edition version 4.3.1.3 and above) is issued to a node, it causes the node to be "quiesced" in the next cluster rebalance.

During rebalance, a quiesced node behaves in some ways as if it has been removed from the cluster, but in other ways as if it is still in the cluster.

Rebalance puts the quiesced node at the end of every partition's "succession list". This can cause a quiesced node that was previously master to handoff master status to a normal (not quiesced) node. This handoff happens only if the normal node and quiesced node have matching full partition versions. Such a handoff is the goal of using quiescence in one particular use case (see below).

The quiesced node at the end of the succession list is excluded from certain algorithms during rebalance: AP rack-aware, AP uniform balance, and 
[SC](/docs/reference/configuration/#strong-consistency) "second phase" rack-aware -- for these purposes the quiesced node behaves as if it is not in the cluster.

Otherwise, the quiesced node behaves as if it is in the cluster -- it will accept transactions, schedule migrations if appropriate, and for [SC](/docs/reference/configuration/#strong-consistency) it will count towards determining that a partition is available.

One last way a quiesced node behaves differently is that it will never drop its data, even if all migrations complete and leave it with a "superfluous" partition version. The assumption is that the quiesced node will be taken down and then return to its prior place as a replica -- keeping data means it will not take as long to re-sync, needing only a "delta" migration instead of a "fill" migration.

### Using Quiescence for Smooth Master Handoff

A common use for quiescence is to enable smooth master handoffs.

Normally, when a node is removed from a cluster, it takes a couple seconds for the remaining nodes to re-cluster and determine a new master for the partitions that had been master on the missing node. During this time, transactions (with timeouts shorter than the "master gap") that are looking for a master node will not find one, and will time out -- i.e. writes, and [SC](/docs/reference/configuration/#strong-consistency) reads. AP reads will by default retry against a replica.

Quiescence can fill this gap -- if the node to be removed is first quiesced, and a rebalance triggered, master handoff will occur during the rebalance, yet the quiesced node will continue to receive transactions (and proxy them to the new master) until all the clients have discovered the new master and moved to it. Once this has happened, the quiesced node can be taken down. The re-clustering caused by this will not have a "master gap". Therefore, the burst of timeouts should instead become a burst of proxies (refer to the proxy related metrics such as [`client_proxy_complete`](/docs/reference/metrics/#client_proxy_complete)).

### Here is procedure to do this during a rolling upgrade

{{#steps}}

{{#steps-step 1 "Ensure the cluster is stable |  `asinfo -v 'cluster-stable:...'`" markdown=true}}

Using the [`cluster-stable`](/docs/reference/info/#cluster-stable) info command, ensure there are no migrations and all the nodes are in the cluster.

```
Admin> asinfo -v 'cluster-stable:size=4;ignore-migrations=no'
aero-cluster1_4:3000 (10.0.3.41) returned:
721D40EF7964

aero-cluster1_2:3000 (10.0.3.224) returned:
721D40EF7964

aero-cluster1_1:3000 (10.0.3.196) returned:
721D40EF7964

aero-cluster1_3:3000 (10.0.3.149) returned:
721D40EF7964
```

{{/steps-step}}

{{#steps-step 2 "Issue the quiesce command |  `asinfo -v 'quiesce:'`" markdown=true}}

The [`quiesce`](/docs/reference/info/#quiesce) command can be issued from [`asadm`](/docs/tools/asadm/index.html) or directly through [`asinfo`](/docs/tools/asinfo/index.html). If using [`asadm`](/docs/tools/asadm/index.html), it should be directed to the node to be quiesced using the `with` modifier, specifying the IP address or node ID of the node to be quiesced:

```
Admin> asinfo -v 'quiesce:' with 10.0.3.224
aero-cluster1_2:3000 (10.0.3.224) returned:
ok
```

Verify the command has been successful by checking the [`pending_quiesce`](/docs/reference/metrics/#pending_quiesce) statistic:

```
Admin> show statistics like pending_quiesce
                            bar Namespace Statistics (2018-10-25 23:33:36 UTC)
NODE           :   aero-cluster1_1:3000   aero-cluster1_2:3000   aero-cluster1_3:3000   aero-cluster1_4:3000
pending_quiesce:   false                  true                   false                  false

                            test Namespace Statistics (2018-10-25 23:33:36 UTC                            
NODE           :   aero-cluster1_1:3000   aero-cluster1_2:3000   aero-cluster1_3:3000   aero-cluster1_4:3000
pending_quiesce:   false                  true                   false                  false
```

{{#note}}
Delaying fill migrations is a good practice in many common situations (independent of quiescing). Refer to the [`migrate-fill-delay`](/docs/reference/configuration/#migrate-fill-delay) configuration parameter for details.
{{/note}}

{{#info}}
If the [`quiesce`](/docs/reference/info/#quiesce) command has been issued to the wrong node, the [`quiesce-undo`](/docs/reference/info/#quiesce-undo) command can be used to 
revert it.
{{/info}}

**Note:** If the [`quiesce`](/docs/reference/info/#quiesce) command is inadvertently issued against all the nodes in the cluster, the subsequent 
[`recluster`](/docs/reference/info/#recluster) command will be ignored:
```
WARNING (partition): (partition_balance_ee.c:435) {test} can't quiesce all nodes - ignoring
```

{{/steps-step}}

{{#steps-step 3 "Issue a recluster command | `asinfo -v 'recluster:'`" markdown=true}}

This is the step where **quiesced masters will handoff to other nodes** and migrations will start.


Issue a [`recluster`](/docs/reference/info/#recluster) command:

```
Admin> asinfo -v 'recluster:'
aero-cluster1_4:3000 (10.0.3.41) returned:
ok

aero-cluster1_2:3000 (10.0.3.224) returned:
ignored-by-non-principal

aero-cluster1_1:3000 (10.0.3.196) returned:
ignored-by-non-principal

aero-cluster1_3:3000 (10.0.3.149) returned:
ignored-by-non-principal
```


Verify command was successful by checking the [`effective_is_quiesced`](/docs/reference/metrics/#effective_is_quiesced) and 
[`nodes_quiesced`](/docs/reference/metrics/#nodes_quiesced) statistics:

```
Admin> show statistics like quiesce
                            bar Namespace Statistics (2018-10-25 23:36:50 UTC                            
NODE                 :   aero-cluster1_1:3000   aero-cluster1_2:3000   aero-cluster1_3:3000   aero-cluster1_4:3000
effective_is_quiesced:   false                  true                   false                  false
nodes_quiesced       :   1                      1                      1                      1
pending_quiesce      :   false                  true                   false                  false

                            test Namespace Statistics (2018-10-25 23:36:50 UTC                            
NODE                 :   aero-cluster1_1:3000   aero-cluster1_2:3000   aero-cluster1_3:3000   aero-cluster1_4:3000
effective_is_quiesced:   false                  true                   false                  false
nodes_quiesced       :   1                      1                      1                      1
pending_quiesce      :   false                  true                   false                  false
```

{{/steps-step}}

{{#steps-step 4 "Check that no more transactions are hitting the quiesced node" markdown=true}}

A few seconds should be enough to be sure all clients have "moved" from the quiesced node to the new masters.

The [`asadm`](/docs/tools/asadm/index.html) `show latency` command can be used to check that the read and write throughput to the quiesced node is down to zero. Usual metrics can be used to verify other type of transactions have also stopped against the node to be quiesced.

```
Admin> show latency
                read Latency (2018-10-25 23:49:19 UTC
                Node                 Time   Ops/Sec   %>1Ms   %>8Ms   %>64Ms
                   .                 Span         .       .       .        .
aero-cluster1_1:3000   23:49:08->23:49:18       9.3     0.0     0.0      0.0
aero-cluster1_2:3000   23:49:08->23:49:18       0.0     0.0     0.0      0.0
aero-cluster1_3:3000   23:49:08->23:49:18       7.5     0.0     0.0      0.0
aero-cluster1_4:3000   23:49:08->23:49:18       8.4     0.0     0.0      0.0
Number of rows: 1

                write Latency (2018-10-25 23:49:19 UTC
                Node                 Time   Ops/Sec   %>1Ms   %>8Ms   %>64Ms
                   .                 Span         .       .       .        .
aero-cluster1_1:3000   23:49:08->23:49:18       2.0     0.0     0.0      0.0
aero-cluster1_2:3000   23:49:08->23:49:18       0.0     0.0     0.0      0.0
aero-cluster1_3:3000   23:49:08->23:49:18       3.2     0.0     0.0      0.0
aero-cluster1_4:3000   23:49:08->23:49:18       4.5     0.0     0.0      0.0
Number of rows: 1
```

{{#note}}
There would typically be a second or two of proxy transactions on the node that was quiesced as clients retrieve the updated partition map and start directing transactions to the new master nodes for the partitions previously owned by the quiesced nodes. It is a good practice to also monitor for proxies transactions to stop on the quiesced node prior to shutting it down. Simply monitor the proxy transactions on the [client transaction metric log line](/docs/reference/serverlogmessages/index.html#ll9a3b0Vu1iwvAekavcMJp2BsGc.) or, alternatively, dynamically enable [proxy](/docs/operations/monitor/latency/index.html#proxy) histogram and monitor their throughput using the [Log Latency Tool](/docs/tools/asloglatency/index.html).
{{/note}}

{{/steps-step}}


{{#steps-step 5 "Take down the quiesced node and proceed with the upgrade or maintenance" markdown=true}}

```
$ sudo systemctl stop aerospike
```

Verify the node has stopped and the cluster is now showing one less node:

```
Admin> info network
                                                       Network Information (2018-10-26 00:02:40 UTC)
                Node               Node                Ip       Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime
                   .                 Id                 .           .      Size            .            Key   Integrity                 .    Conns          .
aero-cluster1_1:3000   BB9211EF53E1600    10.0.3.196:3000   E-4.3.1.4         3      0.000     AFDB3D446F76   True        BB9FD77A03E1600        2   01:33:20
aero-cluster1_3:3000   BB906A7363E1600    10.0.3.149:3000   E-4.3.1.4         3      0.000     AFDB3D446F76   True        BB9FD77A03E1600        2   01:33:20
aero-cluster1_4:3000   *BB9FD77A03E1600   10.0.3.41:3000    E-4.3.1.4         3      0.000     AFDB3D446F76   True        BB9FD77A03E1600        3   01:33:20
Number of rows: 3
```

The [`nodes_quiesced`](/docs/reference/metrics/#nodes_quiesced) statistic is now back to 0:

```
Admin> show stat like quiesce
                     bar Namespace Statistics (2018-10-26 00:01:17 UTC
NODE                 :   aero-cluster1_1:3000   aero-cluster1_3:3000   aero-cluster1_4:3000
effective_is_quiesced:   false                  false                  false
nodes_quiesced       :   0                      0                      0
pending_quiesce      :   false                  false                  false

                     test Namespace Statistics (2018-10-26 00:01:17 UTC
NODE                 :   aero-cluster1_1:3000   aero-cluster1_3:3000   aero-cluster1_4:3000
effective_is_quiesced:   false                  false                  false
nodes_quiesced       :   0                      0                      0
pending_quiesce      :   false                  false                  false
```

**Proceed with the upgrade or other maintenance needed.**

{{/steps-step}}


{{#steps-step 6 "Bring the quiesced node back up" markdown=true}}


Bring the quiesced node back up, and make sure it joins the cluster:

```
$ sudo systemctl start aerospike
```

```
Admin> info
                                                       Network Information (2018-10-26 00:05:49 UTC
                Node               Node                Ip       Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime
                   .                 Id                 .           .      Size            .            Key   Integrity                 .    Conns          .
aero-cluster1_1:3000   BB9211EF53E1600    10.0.3.196:3000   E-4.3.1.4         4      1.015 K   5EDF7C44A664   True        BB9FD77A03E1600        2   01:36:30
aero-cluster1_2:3000   BB912EA003E1600    10.0.3.224:3000   E-4.3.1.4         4      2.798 K   5EDF7C44A664   True        BB9FD77A03E1600        2   00:00:04
aero-cluster1_3:3000   BB906A7363E1600    10.0.3.149:3000   E-4.3.1.4         4    828.000     5EDF7C44A664   True        BB9FD77A03E1600        2   01:36:30
aero-cluster1_4:3000   *BB9FD77A03E1600   10.0.3.41:3000    E-4.3.1.4         4    953.000     5EDF7C44A664   True        BB9FD77A03E1600        1   01:36:30
Number of rows: 4
```

{{/steps-step}}

{{#steps-step 7 "Wait for migrations to complete | `asinfo -v 'cluster-stable:...`" markdown=true}}

Make sure migrations have completed prior to moving on to the next node.  This is done using the [cluster-stable](/docs/reference/info/index.html#cluster-stable) command.  The command should be run on each node in the cluster.  The command should return the same cluster key for every node in the cluster.  The [cluster-stable](/docs/reference/info/index.html#cluster-stable) can be scripted and the results compared programmatically.

**For most common cases, migrations at this point would only consist of [lead migrations](/docs/reference/metrics/?show-removed=1#migrate_tx_partitions_lead_remaining) 
and the time required for completion would be proportional to how long the node has been quiesced.** 

For situations where the stored data is deleted, or when there is no persisted storage, migrations would need to repopulate the data on the node, which would usually 
take longer. A [cold restart](/docs/operations/manage/aerospike/cold_start/) occurring could also have impact on how long migrations would take as a node would take longer to cold restart and could also, in some cases, resurrect previously deleted records.

```
Admin> asinfo -v 'cluster-stable:size=4;ignore-migrations=no'
aero-cluster1_4:3000 (10.0.3.41) returned:
5EDF7C44A664

aero-cluster1_2:3000 (10.0.3.224) returned:
5EDF7C44A664

aero-cluster1_3:3000 (10.0.3.149) returned:
5EDF7C44A664

aero-cluster1_1:3000 (10.0.3.196) returned:
5EDF7C44A664
```


{{#note}}
Waiting for migrations to complete in this step is necessary to ensure that when quiescing the next node, the master ownership change will happen as soon as the recluster is done. Quiescing the next node without waiting for migrations to complete would be also possible but then requires to wait for migrations 
to complete prior to shutting the node to avoid abrupt client and fabric connections cut offs.
{{/note}}


{{/steps-step}}


{{#steps-step 8 "Move to the next node" markdown=true}}

Repeat those steps on the next node.

{{/steps-step}}
{{/steps}}

{{#note}}
**Using Quiescence for Extra Durability**<br><br>

Quiescence may also be used to provide extra durability in various scenarios.

For example, in an AP cluster (replication factor 2) in which a node must be taken down, if it is quiesced, a rebalance triggered, and migrations do complete before removing the quiesced node, two full copies will be present when the node is removed.  The cluster will then still have all data available if another node accidentally goes down before the first node returns.
{{/note}}

{{#note}}
**Quiescing multiple nodes**<br><br>

Quiescing multiple nodes at once can be useful in [rack-aware](/docs/operations/configure/network/rack-aware/index.html) clusters. In such cases, 
quiescing a whole rack at once can speed up maintenance procedures.

In [`strong-consistency`](/docs/reference/configuration/index.html#strong-consistency) enabled namespaces, quiescing [`replication-factor`](/docs/reference/configuration/index.html#replication-factor) number of nodes or more will force masterhood handover (after migrations complete) but will result in unavailability when the nodes are eventually shut down (unless if all nodes quiesced are in the same rack).
{{/note}}

{{#note}}
**Quiescing nodes for namespaces with replication factor 1**<br><br>

For namespaces configured with [`replication-factor`](/docs/reference/configuration/?show-removed=1#replication-factor) `1`, it is necessary to wait for migrations to complete prior to shutting down the server in order for the single copy of each partition owned by that node to have migrated to another node.
{{/note}}

