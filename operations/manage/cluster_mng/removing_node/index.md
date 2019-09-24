---
title : Removing a node
---

##Context

Removing a node from a cluster is as easy as stopping the node, but it is important to follow the steps outlined to ensure proper operation of the cluster and its tools in the long run. Examples:
 * Prevent the remaining nodes in the cluster to re-connect to the removed node if they are restarted.
 * Prevent a cluster from attempting to join another cluster when one of its previously removed node is recommissioned.

{{#note}}
It is a good practice to quiesce a node prior to shutting it down or removing it from a cluster. Refer to the 
[Quiesce Node](/docs/operations/manage/cluster_mng/quiescing_node) page for further details.
{{/note}}

{{#note}}
If the node is shipping records via XDR, it is a good practice to wait for [`xdr_timelag`](/docs/reference/metrics/index.html#xdr_timelag) to drop to zero prior to removing the node from the cluster.
{{/note}}

##Method
{{#steps}}

{{#steps-step 1 "Ensure there are no ongoing migrations" markdown=true}}

{{#note}}
As of version 4.3, the [`cluster-stable`](/docs/reference/info/#cluster-stable) info command can be used in lieu of manually checking the statistics listed below.
{{/note}}

There are several ways to know the cluster migrations state. One of the way is to look at the migration-related statistics Pending Migrates (tx%,rx%) on all the nodes.
**Example: on a 3 node cluster**

```asciidoc
Admin> info namespace
Namespace                         Node   Avail%   Evictions                 Master                Replica     Repl     Stop     Pending       Disk    Disk     HWM        Mem     Mem    HWM      Stop   
        .                            .        .           .   (Objects,Tombstones)   (Objects,Tombstones)   Factor   Writes    Migrates       Used   Used%   Disk%       Used   Used%   Mem%   Writes%   
        .                            .        .           .                      .                      .        .        .   (tx%,rx%)          .       .       .          .       .      .         .   
test        10.0.0.100:3000              N/E        0.000     (0.000  ,0.000  )      (0.000  ,0.000  )      2        false    (0,0)            N/E   N/E     50      0.000 B    0       60     90        
test        10.0.0.103:3000              N/E        0.000     (0.000  ,0.000  )      (0.000  ,0.000  )      2        false    (0,0)            N/E   N/E     50      0.000 B    0       60     90        
test        10.0.0.101:3000              N/E        0.000     (0.000  ,0.000  )      (0.000  ,0.000  )      2        false    (0,0)            N/E   N/E     50      0.000 B    0       60     90        
test                                                0.000     (0.000  ,0.000  )      (0.000  ,0.000  )                        (0,0)       0.000 B                    0.000 B                             
Number of rows: 4
```

For server versions 3.11 and above, make sure the "migrate_partitions_remaining" statistic shows 0 for each node:

``` 
Admin> show statistics like migrate
NODE                        :   10.0.0.100:3000   10.0.0.103:3000   10.0.0.101:3000   
migrate_allowed             :   true              true              true                            
migrate_partitions_remaining:   0                 0                 0                               

```

Another way to monitor the status of migrations is described on the following document:  
https://www.aerospike.com/docs/operations/manage/migration#monitoring-migrations

For versions prior to 3.11, relevant statistics are described on this page:
https://discuss.aerospike.com/t/faq-monitoring-migrations-on-a-live-aerospike-cluster/3200
{{/steps-step}}

{{#steps-step 2 "Shutdown the node" markdown=true}}

Shutdown the node gracefully by stopping the aerospike daemon
```asciidoc
sudo service aerospike stop
```
The shutdown is successful when we see the following message is logged
```asciidoc
finished clean shutdown – exiting
```
The Shutdown can also be ensured by observing the status of Aerospike daemon using the following command
```asciidoc
sudo service aerospike status
* aerospike is not running
```
{{/steps-step}}

{{#steps-step 3 "Update configuration (on all other nodes in the cluster)" markdown=true}}

**Update configuration (on all other nodes in the cluster)**
If this node is in the seed list of other nodes, the configuration of all the other nodes should be updated to ensure that they do not try to connect to this node if they are restarted.

Default location of a configuration file
```asciidoc
/etc/aerospike/aerospike.conf
```
In our example:
On node: 10.0.0.101 comment out the removed node's address(10.0.0.100) from the seed list
```asciidoc

network {
        service {
                address any
 		 access-address 10.0.0.101
                port 3000
        }

        heartbeat {
                mode mesh
                address 10.0.0.101
                port 3002 # Heartbeat port for this node.

                # List one or more other nodes, one ip-address & port per line:
                # mesh-seed-address-port 10.0.0.100  3002
                mesh-seed-address-port 10.0.0.101  3002
		mesh-seed-address-port 10.0.0.103  3002

                interval 250
                timeout 10
        }
```
On node: 10.0.0.103 comment out the removed node's address(10.0.0.100) from the seed list
```asciidoc
network {
        service {
                address any
                access-address 10.0.0.103
                port 3000
        }
        heartbeat {
                mode mesh
                address 10.0.0.103
                port 3002 # Heartbeat port for this node.

                # List one or more other nodes, one ip-address & port per line:
                # mesh-seed-address-port 10.0.0.100 3002
                mesh-seed-address-port 10.0.0.101 3002
                mesh-seed-address-port 10.0.0.103 3002

                interval 250
                timeout 10
        }

```
{{/steps-step}}

{{#steps-step 4 "Tip clear" markdown=true}}

**Multicast mode**
Skip this step to go to step 5
**Mesh mode**
If the cluster is formed using mesh mode, the next step is to run the 'tip-clear' command on all the remaining nodes in the cluster. This is to clear IP or hostname tip list from mesh-mode heartbeat list to prevent the remaining nodes from continuously trying to send heartbeats to the removed node.
```asciidoc
asadm -e "asinfo -v 'tip-clear:host-port-list=XX.XX.XX.XX:3002'"
```
Where XX.XX.XX.XX is the IP address of the node(s) to be removed.

In our example, from one of the nodes, issue the command from within asadm:
```asciidoc
asadm
Admin> asinfo -v 'tip-clear:host-port-list=10.0.0.100:3002'
10.0.0.101:3000 (10.0.0.101) returned:
ok

10.0.0.103:3000 (10.0.0.103) returned:
ok

```

To validate tip-clear, run the following command to log the heart-beat dump in the log file located at /var/log/aerospike/aerospike.log. The heartbeat dump should not contain the node that is decomissioned.
```asciidoc
asadm
Admin> asinfo -v 'dump-hb:verbose=true'
```
In our example
On node 10.0.0.101
```asciidoc
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2605) Heartbeat Dump:
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2616) HB Mode: mesh (2)
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2618) HB Addresses: {10.0.0.101:3002}
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2619) HB MTU: 1500
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2621) HB Interval: 250
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2622) HB Timeout: 10
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2623) HB Fabric Grace Factor: -1
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:2626) HB Protocol: v2 (4)
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:8447) HB Mesh Node (seed): Node: bb9235677270008, Status: active, Last updated: 32223653, Endpoints: {10.0.0.103:3002}
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:6196) HB Channel Count 1
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:6181) HB Channel (mesh): Node: bb9235677270008, Fd: 65, Endpoint: 10.0.0.103:3002, Polarity: inbound, Last Received: 44236581
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:9947) HB Adjacency Size: 1
Jan 23 2017 15:02:21 GMT-0800: INFO (hb): (hb.c:9933) HB Adjacent Node: Node: bb9235677270008, Protocol: 26723, Endpoints: {10.0.0.103:3002}, Last Updated: 44236581
```
On node 10.0.0.103
```asciidoc
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2605) Heartbeat Dump:
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2616) HB Mode: mesh (2)
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2618) HB Addresses: {10.0.0.103:3002}
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2619) HB MTU: 1500
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2621) HB Interval: 250
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2622) HB Timeout: 10
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2623) HB Fabric Grace Factor: -1
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:2626) HB Protocol: v2 (4)
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:8447) HB Mesh Node (seed): Node: bb90a09e3270008, Status: active, Last updated: 32188503, Endpoints: {10.0.0.101:3002}
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:6196) HB Channel Count 1
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:6181) HB Channel (mesh): Node: bb90a09e3270008, Fd: 60, Endpoint: 10.0.0.101:3002, Polarity: outbound, Last Received: 44510068
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:9947) HB Adjacency Size: 1
Jan 23 2017 23:07:23 GMT: INFO (hb): (hb.c:9933) HB Adjacent Node: Node: bb90a09e3270008, Protocol: 26723, Endpoints: {10.0.0.101:3002}, Last Updated: 44510068
```
{{/steps-step}}

{{#steps-step 5 "Alumni reset" markdown=true}}

As a final step, remove the node from the alumni list. The alumni list is used by some tools to refer to all nodes in a cluster, even nodes that may have split from the cluster, so it is important to also clear this node from the list. This command should be run on all the remaining nodes in the cluster:
```asciidoc
asinfo -v 'services-alumni-reset'
```

From asadm:
```asciidoc
asadm
Admin> asinfo -v 'services-alumni-reset'
10.0.0.101:3000 (10.0.0.101) returned:
ok

10.0.0.103:3000 (10.0.0.103) returned:
ok
```
{{/steps-step}}
{{/steps}}
{{#note}}
Notes: Please note that steps 2, 3, 4 and 5 should be run on each of the remaining nodes in the cluster, which 
will be done automatically when issuing the asinfo commands from within asadm.
{{/note}}

{{#info}}
If you want to take down multiple nodes from the cluster, make sure that you start from step 1 and take one node down at a time, waiting for migrations to complete between each node to avoid losing any data.
{{/info}}

{{#note}}
**Note:** It is expected to see clients either timeout or have higher retries to the removed node until the client retrieves an updated partition map, especially for write transactions that typically do not have the option to fall back to a replica. By default, it would take a cluster up to 1.5 seconds to detect that a node has left (based on the configured heartbeat [interval](/docs/reference/configuration#interval) and [timeout](/docs/reference/configuration#timeout)), another 1 or 2 seconds for the cluster to reform and clients by default tend for the partition map every 1 second, it would therefore usually take up to around 5 seconds for the clients to start issuing transactions that were previously targetted to the removed node against the new owner(s) for the partitions the departed noded owned.
{{/note}}

