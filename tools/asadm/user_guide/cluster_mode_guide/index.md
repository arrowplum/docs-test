---
title: Aerospike Admin Cluster Mode Commands
description: Introduction to Aerospike Admin Cluster mode commands.

---

### Help
The `help` command provides a brief description of all supported commands. You can
provide the name of a command to print help for that specific command.

In the example below, we request help for the `info` command.

```asciidoc
Admin> help info
The "info" command provides summary tables for various aspects
of Aerospike functionality.
Modifiers: with
Default: Displays network, namespace, and XDR summary information.
  - dc:
    Displays summary information for each datacenter.
  - namespace:
    The "namespace" command provides summary tables for various aspects
    of Aerospike namespaces.
    Modifiers: with
    Default: Displays usage and objects information for namespaces
      - object:
        Displays object information for each namespace.
      - usage:
        Displays usage information for each namespace.
  - network:
    Displays network information for Aerospike.
  - set:
    Displays summary information for each set.
  - sindex:
    Displays summary information for Secondary Indexes (SIndex).
  - xdr:
    Displays summary information for Cross Datacenter
    Replication (XDR).
```

### Info
Commands within `info` provide diagnostic information in a concise tabular
format. Without additional arguments `info` will execute **network**,
**namespace**, and **xdr** sub-commands. Command descending from
`info` will alert you of potential cluster issues by coloring suspicious test
red. You will also notice that one node name is always green, this node is the
node expected to be the Paxos Principal node. For **namespace** and **set**
subcommands, it displays extra rows in blue, which has sum of statistics per
namespace and set.

#### Info network
The `info network` command primarily serves as a translation table between
Node name, Node ID, and IP. It also provides cluster stats such as the
cluster size, cluster key, number of client connections and uptime for each server.
Also it displays cluster name for server 3.10 and above.

Under the Node Id column **asadm** also indicate the node which is expected to be
the Paxos Principal with an asterisks.

Example Output:
```asciidoc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             Node               Node                  Ip        Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime   
                .                 Id                   .            .      Size            .            Key   Integrity                 .    Conns          .   
172.28.128.3:3000   *BB9635C2A270008   172.28.128.3:3000   E-3.15.0.2         1      0.000     1CEC1DC68B04   True        BB9635C2A270008        5   01:23:56 
Number of rows: 1
```

#### Info namespace
The `info namespace` command displays a summary of important namespace
statistics for each namespace defined on each node ordered by Namespace and
Node. It displays extra row in blue which has sum of some of the statistics.

From 0.1.14 onwards it displays information in two separate tables:

1. Usage: Namespace usage related details
2. Object: Namespace object related details

Example Output:
```asciidoc
Admin> info namespace
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Usage Information (2019-04-02 13:58:18 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace              Node     Total   Expirations,Evictions     Stop         Disk    Disk     HWM   Avail%        Mem     Mem    HWM      Stop      PI           PI      PI     PI
        .                 .   Records                       .   Writes         Used   Used%   Disk%        .       Used   Used%   Mem%   Writes%    Type         Used   Used%   HWM%
bar         172.17.0.6:3000   0.000     (0.000,  0.000)         false           N/E   N/E     50      N/E      0.000 B    0       60     90        shmem     0.000 B    N/E     N/E
bar                           0.000     (0.000,  0.000)                    0.000 B                             0.000 B                                       0.000 B
test        172.17.0.6:3000   3.000     (0.000,  0.000)         false    288.000 B    1       50      99       0.000 B    0       60     90        flash   192.000 B    0       80
test                          3.000     (0.000,  0.000)                  288.000 B                             0.000 B                                     192.000 B
Number of rows: 4

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Object Information (2019-04-02 13:58:18 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace              Node     Total     Repl                      Objects                   Tombstones             Pending   Rack
        .                 .   Records   Factor   (Master,Prole,Non-Replica)   (Master,Prole,Non-Replica)            Migrates     ID
        .                 .         .        .                            .                            .             (tx,rx)      .
bar         172.17.0.6:3000   0.000     1        (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)     0
bar                           0.000              (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)
test        172.17.0.6:3000   3.000     1        (3.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)     0
test                          3.000              (3.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)
Number of rows: 4

```

#### Info set
###### (**Introduced:** 0.0.15)
The `info set` command displays a summary of important set
statistics for each set defined on each namespace on all nodes ordered by Set and
Namespace. It displays extra row in blue which has sum of some of the statistics.

Example Output:
```asciidoc
Admin> info set
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Set Information (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Set   Namespace                  Node      Set        Mem     Objects     Stop    Disable           Set   
      .           .                     .   Delete       used           .   Writes   Eviction        Enable   
      .           .                     .        .          .           .    Count          .           Xdr   
testset   temp        172.16.245.231:3000   false    1.723 KB    36.000     0        false      use-default   
testset   temp        172.16.245.232:3000   false    1.627 KB    34.000     0        false      use-default   
testset   temp        172.16.245.233:3000   false    1.436 KB    30.000     0        false      use-default   
testset   temp                                       4.785 KB   100.000                                       
Number of rows: 4
```

Further statistics for a set can be displayed using the `show statistics` command for specific sets:

```asciidoc
 Admin> show statistics sets for test1 testset
 ```


#### Info sindex
The `info sindex` command displays a summary of important sindex
statistics for each sindex defined on each namespace on all nodes ordered by Sindex and
Node.

Example Output:
```asciidoc
Admin> info sindex
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Secondary Index Information (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                    Node      Index   Namespace       Set       Bin   State     Sync    Keys   Entries   si_accounted_memory      q       w   d   s   
                       .       Name           .         .      Type       .    State       .         .                     .      .       .   .   .   
u10.aerospike.local:3000   int1_idx   test        new_set   NUMERIC      RW   synced   66166   66166                 2186744    998   66166   0   0   
u12.aerospike.local:3000   int1_idx   test        new_set   NUMERIC      RW   synced   67199   67199                 2228896   1000   67199   0   0   
u13.aerospike.local:3000   int1_idx   test        new_set   NUMERIC      RW   synced   66635   66635                 2198616    999   66635   0   0   
u10.aerospike.local:3000   str1_idx   test        new_set    STRING      RW   synced   66166   66166                 3015440    998   66166   0   0   
u12.aerospike.local:3000   str1_idx   test        new_set    STRING      RW   synced   67199   67199                 3061864   1000   67199   0   0   
u13.aerospike.local:3000   str1_idx   test        new_set    STRING      RW   synced   66635   66635                 3026968   1000   66635   0   0
Number of rows: 6
```

Further statistics for a secondary index can be displayed using the `show statistics` command for specific sindex:

```asciidoc
 Admin> show statistics sindex for test test_str_idx
 ```


#### Info xdr
The `info xdr` command shows the current performance characteristics of XDR on each node.

Example Output:
```asciidoc
Admin> info xdr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~XDR Information (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
                    Node     Build        Data    Free          Lag           Req       Req        Req          Cur       Avg      Xdr   
                       .         .     Shipped   Dlog%        (sec)   Outstanding   Shipped    Shipped   Throughput   Latency   Uptime   
                       .         .           .       .            .             .   Success     Errors            .      (ms)        .
u10.aerospike.local:3000    3.3.24   87.688 GB     100   0.000 secs       1.900 K  81.284 M    0.000           1078         6    79170
u12.aerospike.local:3000    3.3.24   87.218 GB     100   0.000 secs       2.100 K  77.373 M    0.000           1054         6    79748
u13.aerospike.local:3000    3.3.24   90.094 GB     100   0.000 secs       1.000 K  80.162 M    0.000            970         7    80336
Number of rows: 3

```

#### Info dc
###### (**Introduced:** 0.0.16)
The `info dc` command displays a summary of important datacenter
statistics for each datacenter.

Example Output:
```asciidoc
Admin> info dc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~DC Information (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                    Node            DC     DC   Namespaces        Lag   Records       Avg         Status   
                       .             .   size            .      (sec)   Shipped   Latency              .   
                       .             .      .            .          .         .      (ms)              .   
u10.aerospike.local:3000   REMOTE_DC_1      0   bar,test     00:00:00         0         0   CLUSTER_DOWN   
u10.aerospike.local:3000   REMOTE_DC_2      0   test         00:00:00         0         0   CLUSTER_DOWN   
u12.aerospike.local:3000   REMOTE_DC_1      0   bar,test     00:00:00         0         0   CLUSTER_DOWN   
u12.aerospike.local:3000   REMOTE_DC_2      0   test         00:00:00         0         0   CLUSTER_DOWN
u13.aerospike.local:3000   REMOTE_DC_1      0   bar,test     00:00:00         0         0   CLUSTER_DOWN   
u13.aerospike.local:3000   REMOTE_DC_2      0   test         00:00:00         0         0   CLUSTER_DOWN  
Number of rows: 6

```

### Show
The `show` commands generally provide a very verbose output about the
requested component. They all support the **like** and **with** modifiers.

#### Configuration
The `show config` command is used to display Aerospike configuration
settings. By default the command will list all server configuration parameters for **service**, **network**, and **namespace**
but you may also provide one of the sub-commands: cluster, namespace, network, service,
xdr, or dc to limit output to those contexts.

In the example below, we request all **network** configuration parameters
containing the words **heartbeat** or **mesh**.
```asciidoc
Admin> show config network like heartbeat mesh
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Configuration (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
NODE                            :   u10.aerospike.local:3000    u12.aerospike.local:3000    u13.aerospike.local:3000
heartbeat.address               :   192.168.120.110             192.168.120.112             192.168.120.113
heartbeat.fabric-grace-factor   :   -1                          -1                          -1
heartbeat.interface-address     :
heartbeat.interval              :   150                         150                         150
heartbeat.mcast-ttl             :   0                           0                           0
heartbeat.mesh-rw-retry-timeout :   500                         500                         500
heartbeat.mesh-seed-address-port:   192.168.120.112             192.168.120.112             192.168.120.112
heartbeat.mode                  :   mesh                        mesh                        mesh
heartbeat.mtu                   :   9001                        9001                        9001
heartbeat.port                  :   3002                        3002                        3002
heartbeat.protocol              :   v2                          v2                          v2
heartbeat.timeout               :   20                          20                          20
```

We can use one more modifier **diff** with `show config`. It gives difference
between configurations in all nodes.

Example Output:
```asciidoc
Admin> show config diff
~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Configuration (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE              :   u10.aerospike.local:3000    u12.aerospike.local:3000    u13.aerospike.local:3000
heartbeat-address :   192.168.120.110             192.168.120.112             192.168.120.113

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Configuration (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                         :   u10.aerospike.local:3000      u12.aerospike.local:3000      u13.aerospike.local:3000
migrate-rx-partitions-initial:   4036                          3904                          3614                            
migrate-tx-partitions-initial:   3362                          4096                          4096                            

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~bar Namespace Configuration (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                         :   u10.aerospike.local:3000      u12.aerospike.local:300       u13.aerospike.local:3000
migrate-rx-partitions-initial:   0                             2752                          0                             
migrate-tx-partitions-initial:   1366                          0                             1386                          

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~temp Namespace Configuration (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                         :   u10.aerospike.local:3000      u12.aerospike.local:3000      u13.aerospike.local:3000
migrate-rx-partitions-initial:   0                             2752                          0                             
migrate-tx-partitions-initial:   1366                          0                             1386 
```

For large clusters we can use `-flip` option to flip output table for simplicity and ease of understanding.

Example Output:
```asciidoc
Admin> show config namespace like partition -flip
~~~~~~~~~~~~~~~~~~~~test Namespace Configuration (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~  
                    NODE   partition-tree-locks   partition-tree-sprigs   sindex.num-partitions   
u10.aerospike.local:3000                      8                      64                      32   
u12.aerospike.local:3000                      8                      64                      32   
u13.aerospike.local:3000                      8                      64                      32   
Number of rows: 3

~~~~~~~~~~~~~~~~~~~~bar Namespace Configuration (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~  
                    NODE   partition-tree-locks   partition-tree-sprigs   sindex.num-partitions   
u10.aerospike.local:3000                      8                      64                      32   
u12.aerospike.local:3000                      8                      64                      32   
u13.aerospike.local:3000                      8                      64                      32   
Number of rows: 3
```

#### Distribution
The `show distribution` displays [histograms](/docs/reference/info#histogram)
and supports object_size and time_to_live histograms. For server version 3.7.5 and below, it displays
eviction histogram also.

For object_size, the `-b` parameter can be used to get bytewise distribution. For server versions 4.1.0.1 and below, 
the `-k` option helps to set the maximum number of buckets to show.

In the below example we can see that 10 percent of our objects are set to expire
in 328320 seconds.

```asciidoc
Admin> show distribution time_to_live 
~~~~~~~~~~~~~~~~~~~~~~~~~~~test - TTL Distribution in Seconds (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                 Percentage of records having ttl less than or equal to value measured in Seconds
                    Node       10%      20%      30%      40%      50%      60%      70%      80%      90%     100%   
u10.aerospike.local:3000    328320   331776   335232   338688   338688   342144   342144   345600   345600   345600  
u12.aerospike.local:3000    328320   331776   335232   338688   338688   342144   342144   345600   345600   345600
u13.aerospike.local:3000    328320   331776   335232   338688   338688   342144   342144   345600   345600   345600
Number of rows: 3
```

#### Latency
The `show latency` command displays [latency](/docs/reference/info#latency) characteristics of reads, writes,
and proxies.

We can get latency for specific time range in intervals by using parameters `-f`, `-d` and `-t`. 
Also we can set `-m` to display latency output machine wise. Default display is histogram name wise.

In the below example we look at the latency of writes_master.

```asciidoc
Admin> show latency like writes_master
~~~~~~~~~~~~~~~writes_master Latency (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~
                    Node                  Time   Ops/Sec  %>1Ms  %>8Ms  %>64Ms
                       .                  Span         .      .      .       .
u10.aerospike.local:3000    08:27:58->08:28:08    2044.7   1.09    0.0     0.0
u12.aerospike.local:3000    08:27:58->08:28:08    2012.6   0.77    0.0     0.0
u13.aerospike.local:3000    08:27:58->08:28:08    1968.9   1.03    0.0     0.0
Number of rows: 3
```

The `show latency` command supports `for` modifier to display namespace wise latency. It also shows aggregate latency for input namespaces (filtered by `for`) in blue. This feature works only for server version 3.9 and above.

The following example shows query latency for _test_ and _bar_ namespaces which got filtered by `for` input (_te_ and _b_).
The third row without namespace name is aggregate latency for test and bar namespace. Though not visible here, this row has blue font. 

```asciidoc
Admin> show latency for te b like query
~~~~~~~~~~~~~~~~~~~~~~~~~~query Latency (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~
                       Node   Namespace                 Time   Ops/Sec  %>1Ms  %>8Ms  %>64Ms
                          .           .                 Span         .      .      .       .   
1.0.0.127.in-addr.arpa:3000   bar         08:27:58->08:28:08     295.2    2.0    0.0     0.0   
1.0.0.127.in-addr.arpa:3000   test        08:27:58->08:28:08     100.0    2.7    0.0     0.0   
1.0.0.127.in-addr.arpa:3000               08:27:58->08:28:08     395.2   2.18    0.0     0.0   
Number of rows: 3
```

#### Mapping
The `show mapping` command displays mapping from IP to Node-ID and Node-ID to IPs.
By default it displays both maps, but
sub-commands ip, and node will confine the output to
a single map. Also we can use `like` modifier to input substring of expected IP or Node-ID.

The following example shows IP to Node-ID mapping for IP which has substring either "231" or "233".

```asciidoc
Admin> show mapping ip like 231 233
~IP to NODE-ID Mapping (2018-03-02 08:28:09 UTC)~
                 IP           NODE-ID   
172.16.245.231:3000   BB9635C2A270008   
172.16.245.233:3000   C81635C2A270008   
Number of rows: 2
```

The following example shows Node-ID to IPs mapping for Node-ID which has substring "BB". 
It displays all available endpoints for Node.

```asciidoc
Admin> show mapping node like BB
~~~~~~~~~~~~~~~~~~NODE-ID to IPs Mapping (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~
        NODE-ID                                                                  IPs   
BB9635C2A270008   172.16.245.231:3000, 1.2.3.4:3000, 1.2.3.5:3000, [2001::1234]:3000   
Number of rows: 1
```

#### Pmap
###### (**Introduced:** 0.1.12)
The `show pmap` command displays partition map analysis of the Aerospike cluster.

The following example shows the output of the `show pmap` command. 

**Primary Partitions:** Total number of primary partitions for a specific namespace on that node. 

**Secondary Partitions:** Total number of secondary partitions for a specific namespace on that node. 

**Missing Partitions:** The partition count which is neither in "sync state" (S) nor in the "desync state" (D), 
for a namespace throughout the cluster are represented as missing partitions. 
In a stable cluster the missing partitions should be zero.  

```asciidoc
Admin> show pmap
~~~~~~~~~~~~~~~~~~~~~Partition Map Analysis (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~  
     Cluster   Namespace                       Node      Primary    Secondary      Missing   
         Key           .                          .   Partitions   Partitions   Partitions   
69579650283B   bar         u10.aerospike.local:3000         1326         1384            0   
69579650283B   bar         u12.aerospike.local:3000         1400         1352            0   
69579650283B   bar         u13.aerospike.local:3000         1370         1360            0   
69579650283B   test        u10.aerospike.local:3000         1326         1384            0   
69579650283B   test        u12.aerospike.local:3000         1400         1352            0   
69579650283B   test        u13.aerospike.local:3000         1370         1360            0   
Number of rows: 6
```


#### Statistics
The `show statistics` command displays all server statistics from several
server components. By default it returns statistics for **bin**, **set**, **service**, and **namespace** but the
sub-commands bins, namespace, service, sets, sindex, xdr, and dc will confine the output to
a single context. Also we can set `-t` parameter to get extra column for total of statistics. Total column displays
sum of statistics with numeric values, for remaining statistics it displays `N/E`.

The following example shows batch api statistics with total column. We can see that each node has
executed more than 700,000 batch requests.

```asciidoc
Admin> show statistics service like batch -t
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Service Statistics (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
NODE                           :   u10.aerospike.local:3000      u12.aerospike.local:3000      u13.aerospike.local:3000    Total   
batch_errors                   :   0                             0                             0                           0
batch_initiate                 :   842926                        826436                        731764                      2401126
batch_queue                    :   0                             0                             0                           0
batch_timeout                  :   0                             0                             0                           0
batch_tree_count               :   0                             0                             0                           0
stat_slow_trans_queue_batch_pop:   4148                          3506                          0                           7654
```

For large clusters we can use `-flip` option to flip output table for simplicity and ease of understanding.

Example Output:
```asciidoc
Admin> show statistics namespace for test like partition-tree -flip
~~~~~~~~~test Namespace Statistics (2018-03-02 08:28:09 UTC)~~~~~~~~~~~
             NODE          partition-tree-locks   partition-tree-sprigs   
u10.aerospike.local:3000                      8                      64   
u12.aerospike.local:3000                      8                      64   
u13.aerospike.local:3000                      8                      64   
Number of rows: 3
```


### Features
###### (**Introduced:** 0.0.15)
The `features` command displays features used in cluster. It supports **like** and **with** modifiers.

Example Output:
```asciidoc
Admin> features
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Features (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE           :   u10.aerospike.local:3000      u12.aerospike.local:3000      u13.aerospike.local:3000   
AGGREGATION    :   NO                            NO                            NO                            
BATCH          :   NO                            NO                            NO          
INDEX-ON-DEVICE:   NO                            NO                            NO   
INDEX-ON-PMEM  :   NO                            NO                            NO                     
KVS            :   YES                           YES                           YES                           
LDT            :   NO                            NO                            NO                            
QUERY          :   YES                           YES                           YES      
RACK-AWARE     :   YES                           YES                           YES      
SC             :   NO                            NO                            NO                             
SCAN           :   NO                            NO                            NO     
SECURITY       :   NO                            NO                            NO                            
SINDEX         :   YES                           YES                           YES   
TLS (FABRIC)   :   NO                            NO                            NO    
TLS (HEARTBEAT):   NO                            NO                            NO    
TLS (SERVICE)  :   NO                            NO                            NO                            
UDF            :   NO                            NO                            NO                            
XDR DESTINATION:   NO                            NO                            NO                            
XDR ENABLED    :   NO                            NO                            NO
```

### Summary
###### (**Introduced:** 0.1.9)
The `summary` command displays summary of cluster. This command accepts remote server credentials to collect system statistics and show them in summary.
By default it collects Aerospike data from all nodes but system statistics only from the localhost (if it is a node of a connected cluster). 
To enable remote system statistics collection, one can use `—-enable-ssh` option. This command accepts more ssh credentials through the following options:
`—-ssh-user`, `—-ssh-pwd`, `—-ssh-port`, and `—-ssh-key`. Also one can provide all credentials through a file 
by using the option `—-ssh-cf`. Refer to `help summary` for further details.

Example Output:
```asciidoc
Admin> summary -l
Cluster
=======

   1.   Server Version     :  E-3.15.1.3
   2.   OS Version         :  CentOS release 6.7 (Final) (2.6.32-573.3.1.el6.x86_64)
   3.   Cluster Size       :  1
   4.   Devices            :  Total 0, per-node 0
   5.   Memory             :  Total 8.000 GB, 0.50% used (40.960 MB), 99.50% available (7.960 GB)
   6.   Disk               :  Total 0.000 B, 0.00% used (0.000 B), 0.00% available contiguous space (0.000 B)
   7.   Usage (Unique Data):  15.000 B  in-memory, 0.000 B  on-disk
   8.   Active Namespaces  :  1 of 2
   9.   Features           :  KVS, Rack-aware, SINDEX


Namespaces
==========

   test
   ====
   1.   Devices            :  Total 0, per-node 0
   2.   Memory             :  Total 4.000 GB, 1.00% used (40.960 MB), 99.00% available (3.960 GB)
   3.   Replication Factor :  2
   4.   Rack-aware         :  True
   5.   Master Objects     :  1.000  
   6.   Usage (Unique Data):  15.000 B  in-memory, 0.000 B  on-disk

   bar
   ===
   1.   Devices            :  Total 0, per-node 0
   2.   Memory             :  Total 4.000 GB, 0.00% used (0.000 B), 100.00% available (4.000 GB)
   3.   Replication Factor :  2
   4.   Rack-aware         :  False
   5.   Master Objects     :  0.000  
   6.   Usage (Unique Data):  0.000 B  in-memory, 0.000 B  on-disk

```

### Collectinfo
The `collectinfo` command collects cluster information (statistics, and configurations) and aerospike conf file for the local node it is run from. 
It also collects system statistics of all nodes if remote server credentials are provided, otherwise it collects system stats for the local node only.

By default `collectinfo` collects Aerospike data from all nodes but system statistics only from localhost (if it is a node of connected cluster). 
To enable remote system statistics collection, one can use the `—-enable-ssh` option. This command accepts more ssh credentials 
through options like `—-ssh-user`, `—-ssh-pwd`, `—-ssh-port`, and `—-ssh-key`. Also user can provide all credentials 
through file by using option `—-ssh-cf`. Please check `help collectinfo` for more details.

### Pager
###### (**Introduced:** 0.0.17)
The `pager` command sets pager for output. For output which can not fit in
output console, this command gives option to scroll each output table vertically as well as
horizontally.

### Other Commands

#### Asinfo
The `asinfo` command provides raw access to Aerospike info protocol. With it
you can change live configurations and view a wide array of technical data for
the cluster. For a list of command strings see
[asinfo documentation](/docs/tools/asinfo/index.html#commands). The asinfo command allows
the user to copy paste commands from the command line asinfo tool and execute
them across the entire cluster.

Unlike the command line tool, to select specific nodes you need to use the
**with** modifier and you can filter the results with the **like** modifier.

The below `asinfo` command will retrieve the configurations from all nodes
and filter for configurations containing the work "batch".

```asciidoc
Admin> asinfo -v get-config like batch
172.16.245.231 (172.16.245.231) returned:
batch-threads=4;batch-max-requests=5000;batch-priority=200;query-batch-size=100

172.16.245.232 (172.16.245.232) returned:
batch-threads=4;batch-max-requests=5000;batch-priority=200;query-batch-size=100

172.16.245.233 (172.16.245.233) returned:
batch-threads=4;batch-max-requests=5000;batch-priority=200;query-batch-size=100

172.16.245.234 (172.16.245.234) returned:
batch-threads=4;batch-max-requests=5000;batch-priority=200;query-batch-size=100
```

#### Watch
The `watch` command should come before another **asadm** command and has two
optional fixed-position arguments. The first position is the number of seconds
to wait between iterations and the second position is the number of iterations
to execute.

The example below will run `info service` 3 times with a 5 second sleep
between iterations. Though not visible here, it will also highlights changes.

```asciidoc
Admin> watch 5 3 info network
[ 2018-03-02 14:07:23 'info network' sleep: 5.0s iteration: 1  of 3 ]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information (2018-03-02 08:37:23 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             Node               Node                  Ip        Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime   
                .                 Id                   .            .      Size            .            Key   Integrity                 .    Conns          .   
172.28.128.3:3000   BB9635C2A270008    172.28.128.3:3000   E-3.99.3.5         2      0.000     D03AB62FF585   True        F3D635C2A270008        3   09:29:30   
172.28.128.3:3900   *F3D635C2A270008   172.28.128.3:3900   C-3.15.0.1         2      0.000     D03AB62FF585   True        F3D635C2A270008        3   10:09:22   
Number of rows: 2


[ 2018-03-02 14:07:28 'info network' sleep: 5.0s iteration: 2  of 3 ]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information (2018-03-02 08:37:28 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             Node               Node                  Ip        Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime   
                .                 Id                   .            .      Size            .            Key   Integrity                 .    Conns          .   
172.28.128.3:3000   BB9635C2A270008    172.28.128.3:3000   E-3.99.3.5         2      0.000     D03AB62FF585   True        F3D635C2A270008        4   09:29:35   
172.28.128.3:3900   *F3D635C2A270008   172.28.128.3:3900   C-3.15.0.1         2      0.000     D03AB62FF585   True        F3D635C2A270008        4   10:09:27   
Number of rows: 2


[ 2018-03-02 14:07:33 'info network' sleep: 5.0s iteration: 3  of 3 ]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information (2018-03-02 08:37:33 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             Node               Node                  Ip        Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime   
                .                 Id                   .            .      Size            .            Key   Integrity                 .    Conns          .   
172.28.128.3:3000   BB9635C2A270008    172.28.128.3:3000   E-3.99.3.5         2      0.000     D03AB62FF585   True        F3D635C2A270008        4   09:29:40   
172.28.128.3:3900   *F3D635C2A270008   172.28.128.3:3900   C-3.15.0.1         2      0.000     D03AB62FF585   True        F3D635C2A270008        4   10:09:32   
Number of rows: 2
```

