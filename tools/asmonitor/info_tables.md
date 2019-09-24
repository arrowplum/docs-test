### Displaying a Single Table

To display the contents of a particular Table, type the info command followed by the table name.  For example, to see the Node table, you would type:
```
info Node
```

{{#note}}
Table names are case-sensitive.
{{/note}}

#### The Node Table

In the asmonitor console, the output from the info command or the info Node command displays a Node table.  An example is:

```
Monitor> info Node
 
 === NODE ===
ip:port                 Build   Cluster      Cluster   Free   Free   Migrates              Node   Replicated    Sys
                            .      Size   Visibility   Disk    Mem          .                id      Objects   Free
                            .         .            .    pct    pct          .                 .            .    Mem
u9.aerospike.local     3.2.8        12         true     76     66      (0,0)   BB99DEC05CA0568    312.118 M     70
u11.aerospike.local    3.2.8        12         true     76     65      (0,0)   BB9500905CA0568    313.077 M     86
u10.aerospike.local    3.2.8        12         true     76     66      (0,0)   BB931F106CA0568    312.246 M     70
Interpreting the Data in the Node Table
```


The ` info Node` command shows the following attributes for each of the nodes:


|Column title|Description|
| --- | --- |
|Host:port| This displays the IP Address or fully qualified domain name of the node along with the info port|
| Build | Aerospike Server version number.  In general, all of the nodes in the cluster should have the same version of the software.|
|Cluster Size|Size of the cluster as determined by this node.  This should be the same value for all nodes in the cluster.|
|Cluster Visibility| Indicates whether this node can connect to all of the other nodes in the cluster. False normally means there is some issue either in configuration/network or the cluster. The way that cluster visibility works is this: Aerospike compares the succession list of each node, with the list of nodes shown in asmonitor. For example, if asmonitor can only connect to 2 nodes and a 3rd node has reached its connection limit, then cluster visibility would show as false.If nodes have multiple network interfaces and all are being exported in the succession list, then the cluster visibility will also be false. To resolve this, fix the configuration file of the node with the multiple interfaces to only export one IP from the succession list.If one node cannot see the others but others can see it, then cluster visibility can also be false.Cluster visibilty is an indication something might be wrong. Also it is always good to run the info command multiple times to confirm if visibility is false several times in a row to make sure the cluster has stabilized.|
|Free Disk %|Percent of SSD space that is still available over all SSDs and persistence files. If you have more than one namespace, you should check the Free Disk % in the Namespace table for more information about the individual namespaces.  We recommend that you target at least 40% of free disk space per namespace.|
|Free Memory %|Percentage of free memory for that node.  We recommend at least 40%.|
|Migrates|Number of migration operations in progress:  (outbound, inbound).  These should both be zero unless a cluster is in the process of rebalancing after a node addition/removal.|
|Node ID|Internal node ID.|
|Replicated Objects|Total number of records on this node, in millions. This includes all records whether this node is the data master or the replica for this data.|
|Sys Free Mem|% of system (operating system) memory that is free.|



The `info Namespace` command shows the following attributes for each of the nodes:


|Column| Description |
|----------|----------------|
|ip/namespace| Node ip and the namespace name on that node|
|Avail pct| Amount of contiguous space available for aerospike to write data. This is more critical than used disk % , since writes will fail if avail pct is zero even if used disk % is not 100 %.|
|Evicted objects| Number of objects evicted due to high water mark|
|Objects|Number of master objects on the node|
|Repl Factor| Replication factor set for the cluster|
|Stop writes| Indicated whether the node is accepting writes or not. If this is true, writes will fail. It can be true due to low (<5%) avail_pct or due to hitting configured stop_writes_pct config.|
|Used Disk| Amount of disk space used in the configured storage device. Will be n/a for in-memory database with no persistance.|
|Used disk %|Percentage of disk space used in the configured storage device wrt total device capacity. Will be n/a for in-memory database with no persistance.|
|Used memory| Amount of memory used up by primary indexes + secondary index if configured + data in memory if configured|
|Used memory %| Amount of used memory as defined above as a percentage of total memory configured for the namespace on the node|
|hwm Disk| high water mark for persistent storage usage as configured for the namespace. Objects will be evicted once used disk % crosses this threshold|
|hwm Mem|high water mark for memory usage as configured for the namespace. Objects will be evicted once used memory % crosses this threshold|

The `info XDR` command shows the following attributes for each of the nodes:


|Column| Description |
|----------|----------------|
|Nodes| Ip or name of the host based upon DNS resolution availability|
|Build| The version of XDR running on the node|
|Bytres shipped| Sum of all packets shipped by XDR. This includes data/object size + network overheads|
|Free dlog| Percentage of digest log available for new operations to be logged This should be non zero in healthy nodes.|
|Lag secs| Time difference between last logged operation and last shipped operation in seconds. This should be close to zero for healthy operations.|
|Req. Outstanding| Number of operations / requests yet to be processed. This should be close to zero in healthy operations.|
|Req. Relog| Number of operations re-logged for processing due to any failure. This should be close to zero in healthy operations.|
|Req. Shipped| Number of operations shipped to remote destination|
|Throughput| Number of operations being shipped per second during last 10  seconds|
|Uptime| Uptime of XDR process on the given node|

The `info SETS` command shows the following attributes for each of the nodes:


|Column| Description |
|----------|----------------|
|ip/ns:set| The ip or name of the node , the namespace name and the set name on the node|
|Delete sets| This shows if the set is being deleted presently. Once a set is deleted, it is removed gradually by nsup|
|Evict hwm| The maximum number of objects in the set after which objects start being evicted.|
|Objects| Number of objects in the set presently|
|Stop writes count| Number of objects after which the set will no longer accept writes. This should be higher than  Evict hwm above.|

Just running the ` info` command will by default run both Node and Namespace tables.
