---
title: Aerospike Admin Collectinfo-Analyzer Mode Commands
description: Introduction to Aerospike Admin Collectinfo-Analyzer mode commands.

---

### Help
The `help` command provides a brief description of all supported commands. You can
provide the name of a command to print help for that specific command.

In the example below, we request help for the `info` command.

```asciidoc
Collectinfo-analyzer> help info
The "info" command provides summary tables for various aspects
of Aerospike functionality.
Default: Displays network, namespace, and xdr summary information.
  - dc:
    Displays datacenter summary information.
  - namespace:
    The "namespace" command provides summary tables for various aspects
    of Aerospike namespaces.
    Default: Displays usage and objects information for namespaces
      - object:
        Displays object information for each namespace.
      - usage:
        Displays usage information for each namespace.
  - network:
    Displays network summary information.
  - set:
    Displays set summary information.
  - sindex:
    Displays secondary index (SIndex) summary information).
  - xdr:
    Displays Cross Datacenter Replication (XDR) summary information.
```

### List
The `list` command displays the collectinfo currently added to collectinfo-analyzer. 

Example Output: 
```asciidoc
Collectinfo-analyzer> list
2017-03-08 07:04:33 UTC: /home/vagrant/shared/data/healthdata/remote_cluster
```

### Info
Commands within `info` provide diagnostic information in a concise tabular
format. Without additional arguments `info` will execute **network**,
**namespace**, and **xdr** sub-commands. Command descending from
`info` will alert you of potential cluster issues by coloring suspicious test
red. If cluster principal and node ids provided in collectinfo, one node name will be green,
this node is the node expected to be the Paxos Principal node. For **namespace** and **set**
subcommands, it displays extra rows in blue, which has sum of statistics per
namespace and set.

In `info` output, some columns might display _N/E_. This is because of the lack of
sufficient information in collectinfo.

#### Info network
The `info network` command primarily serves as a translation table between
Node name, Node ID, and IP. It also provides cluster stats such as the
cluster size, cluster key, number of client connections and uptime for each server.

Under the Node Id column **asadm** also indicate the node which is expected to be
the Paxos Principal with an asterisks.

Example Output:
```asciidoc
Collectinfo-analyzer> info network
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information (2017-11-14 09:03:45 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             Node               Node                  Ip        Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime   
                .                 Id                   .            .      Size            .            Key   Integrity                 .    Conns          .   
172.28.128.3:3000   *BB9635C2A270008   172.28.128.3:3000   E-3.15.0.2         1      0.000     1CEC1DC68B04   True        BB9635C2A270008        3   01:01:18   
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
Collectinfo-analyzer> info namespace
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
The `info set` command displays a summary of important set
statistics for each set defined on each namespace on all nodes ordered by Set and
Namespace. It displays extra row in blue which has sum of some of the statistics.

Example Output:
```asciidoc
Collectinfo-analyzer> info set
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Set Information (2016-03-18 14:21:52 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Set   Namespace                          Node      Set          Mem   Objects     Stop    Disable           Set   
      .           .                             .   Delete         used         .   Writes   Eviction        Enable   
      .           .                             .        .            .         .    Count          .           Xdr   
demoset   test        1.0.0.127.in-addr.arpa:3000   false     48.000 B    1.000     0        false      use-default   
demoset   test                                                48.000 B    1.000                                       
eg-set    test        1.0.0.127.in-addr.arpa:3000   false     14.000 B    1.000     0        false      use-default   
eg-set    test                                                14.000 B    1.000                                       
testset   test        1.0.0.127.in-addr.arpa:3000   false    338.287 MB   5.141 M   0        false      use-default   
testset   test                                               338.287 MB   5.141 M                                     
Number of rows: 6
```

Further statistics for a set can be displayed using the `show statistics` command for specific sets:

```asciidoc
 Collectinfo-analyzer> show statistics sets for test1 testset
 ```

#### Info sindex
The `info sindex` command displays a summary of important sindex
statistics for each sindex defined on each namespace on all nodes ordered by Sindex and
Node.

Example Output:
```asciidoc
Collectinfo-analyzer> info sindex
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Secondary Index Information (2016-03-22 09:45:56 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                       Node           Index   Namespace       Set      Bin   State     Sync   Keys   Entries   si_accounted_memory   q      w     d   s   
                          .            Name           .         .     Type       .    State      .         .                     .   .      .     .   .   
1.0.0.127.in-addr.arpa:3000   test1_str_idx   test        testset   STRING      RW   synced   649    649                     46912   0    649     0   0   
1.0.0.127.in-addr.arpa:3100   test1_str_idx   test        testset   STRING      RW   synced   343    343                     22216   0    343     0   0   
1.0.0.127.in-addr.arpa:3200   test1_str_idx   test        testset   STRING      RW   synced   306    306                     18688   0    649   343   0   
1.0.0.127.in-addr.arpa:3000   test_str_idx    temp        testset   STRING      RW   synced   650    650                     47424   0    817   167   0   
1.0.0.127.in-addr.arpa:3100   test_str_idx    temp        testset   STRING      RW   synced   693    693                     55504   0   1217   524   0   
1.0.0.127.in-addr.arpa:3200   test_str_idx    temp        testset   STRING      RW   synced   657    657                     50440   0   1000   343   0   
Number of rows: 6
```

Further statistics for a secondary index can be displayed using the `show statistics` command for specific sindex:

```asciidoc
 Collectinfo-analyzer> show statistics sindex for test test_str_idx
 ```

#### Info xdr
The `info xdr` command shows the current performance characteristics of XDR on each node.

Example Output:
```asciidoc
Collectinfo-analyzer> info xdr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~XDR Information (2016-03-18 14:21:52 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                       Node                   Build       Data    Free        Lag           Req       Req       Req          Cur       Avg        Xdr   
                          .                       .    Shipped   Dlog%      (sec)   Outstanding   Shipped   Shipped   Throughput   Latency     Uptime   
                          .                       .          .       .          .             .   Success    Errors            .      (ms)          .   
1.0.0.127.in-addr.arpa:3000   3.7.1-xdr-29-gd9b3008   0.000 B      100   00:00:00      94.200 K   0.000     0.000              0         0   00:01:24   
Number of rows: 1

```

#### Info dc
The `info dc` command displays a summary of important datacenter
statistics for each datacenter.

In the example below, DC information for only one snapshot is displayed as second 
collectinfo is created by old version of asadm so it does not have dc stats in it.
```asciidoc
Collectinfo-analyzer> info dc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~DC Information (2016-03-18 14:21:52 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                       Node            DC     DC   Namespaces        Lag   Records       Avg         Status   
                          .             .   size            .      (sec)   Shipped   Latency              .   
                          .             .      .            .          .         .      (ms)              .   
1.0.0.127.in-addr.arpa:3000   REMOTE_DC_1      0   bar          00:00:00         0         0   CLUSTER_DOWN   
Number of rows: 1

```

### Show
The `show` commands generally provide a very verbose output about the
requested component. They all support the **like** modifier.

#### Configuration
The `show config` command is used to display Aerospike configuration
settings. By default the command will list all server configuration parameters for **service**, **network**, and **namespace**
but you may also provide one of the sub-commands: cluster, namespace, network, service,
xdr, or dc to limit output to those contexts.

In the example below, we request all **network** configuration parameters
containing the words **heartbeat** or **mesh**.
```asciidoc
Collectinfo-analyzer> show config network like heartbeat mesh
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Configuration (2016-03-21 12:42:01 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE              :   1.0.0.127.in-addr.arpa:3000   1.0.0.127.in-addr.arpa:3100   1.0.0.127.in-addr.arpa:3200   
heartbeat-address :   10.0.2.15                     10.0.2.15                     10.0.2.15                     
heartbeat-interval:   150                           150                           150                           
heartbeat-mode    :   mesh                          mesh                          mesh                          
heartbeat-port    :   3002                          3102                          3202                          
heartbeat-protocol:   v2                            v2                            v2                            
heartbeat-timeout :   10                            10                            10                            
mesh-address      :   10.0.2.15                     10.0.2.15                     10.0.2.15                     
mesh-port         :   3102                          3002                          3002
```

We can use one more modifier **diff** with `show config`. It gives difference
between configurations in all nodes.

Example Output:
```asciidoc
Collectinfo-analyzer> show config diff
~~~~~~~~~~~~~~~~~~~~~~~~~Service Configuration (2016-03-21 12:42:01 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                   :   1.0.0.127.in-addr.arpa:3000   1.0.0.127.in-addr.arpa:3100   1.0.0.127.in-addr.arpa:3200   
pidfile                :   /var/run/aerospike/asd0.pid   /var/run/aerospike/asd1.pid   /var/run/aerospike/asd2.pid   
object-size-hist-period:   3600                          3600                          7200

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Configuration (2016-03-21 12:42:01 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE             :   1.0.0.127.in-addr.arpa:3000   1.0.0.127.in-addr.arpa:3100   1.0.0.127.in-addr.arpa:3200   
fabric-port      :   3001                          3101                          3201                          
heartbeat-port   :   3002                          3102                          3202                          
mesh-port        :   3102                          3002                          3002                          
network-info-port:   3003                          3103                          3203                          
service-port     :   3000                          3100                          3200                          

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~test Namespace Configuration (2016-03-21 12:42:01 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                         :   1.0.0.127.in-addr.arpa:3000     1.0.0.127.in-addr.arpa:3100   1.0.0.127.in-addr.arpa:3200   
data-in-memory               :   false                           N/E                           N/E                           
defrag-lwm-pct               :   50                              N/E                           N/E                           
defrag-queue-min             :   0                               N/E                           N/E                           
defrag-sleep                 :   1000                            N/E                           N/E                           
defrag-startup-minimum       :   10                              N/E                           N/E                           
file                         :   /opt/aerospike/data/test0.dat   N/E                           N/E                           
filesize                     :   5368709120                      N/E                           N/E                           
flush-max-ms                 :   1000                            N/E                           N/E                           
fsync-max-sec                :   0                               N/E                           N/E                           
max-write-cache              :   67108864                        N/E                           N/E                           
migrate-rx-partitions-initial:   0                               2104                          2730                          
migrate-tx-partitions-initial:   3448                            869                           517                           
min-avail-pct                :   5                               N/E                           N/E 
post-write-queue             :   256                             N/E                           N/E                           
total-bytes-disk             :   5368709120                      N/E                           N/E                           
writecache                   :   67108864                        N/E                           N/E                           
writethreads                 :   1                               N/E                           N/E                           

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~bar Namespace Configuration (2016-03-21 12:42:01 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                         :   1.0.0.127.in-addr.arpa:3000   1.0.0.127.in-addr.arpa:3100   1.0.0.127.in-addr.arpa:3200   
migrate-rx-partitions-initial:   0                             0                             2730                          
migrate-tx-partitions-initial:   1344                          1386                          0                             

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~temp Namespace Configuration (2016-03-21 12:42:01 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                         :   1.0.0.127.in-addr.arpa:3000     1.0.0.127.in-addr.arpa:3100     1.0.0.127.in-addr.arpa:3200     
file                         :   /opt/aerospike/data/temp0.dat   /opt/aerospike/data/temp1.dat   /opt/aerospike/data/temp2.dat   
migrate-rx-partitions-initial:   3262                            3240                            3992                            
migrate-tx-partitions-initial:   3774                            3990                            2730
```

For large clusters we can use `-flip` option to flip output table for simplicity and ease of understanding.

Example Output:
```asciidoc
Collectinfo-analyzer> show config namespace like partitions-remaining -flip
~~~~~~~~~~~~~~~~~~~~~test Namespace Configuration (2016-03-18 14:21:52 UTC)~~~~~~~~~~~~~~~~~~~~  
                       NODE   migrate-rx-partitions-remaining   migrate-tx-partitions-remaining   
1.0.0.127.in-addr.arpa:3000                                 0                                 0   
Number of rows: 1

~~~~~~~~~~~~~~~~~~~~~bar Namespace Configuration (2016-03-18 14:21:52 UTC)~~~~~~~~~~~~~~~~~~~~~
                       NODE   migrate-rx-partitions-remaining   migrate-tx-partitions-remaining   
1.0.0.127.in-addr.arpa:3000                                 0                                 0   
Number of rows: 1

~~~~~~~~~~~~~~~~~~~~~temp Namespace Configuration (2016-03-18 14:21:52 UTC)~~~~~~~~~~~~~~~~~~~~
                       NODE   migrate-rx-partitions-remaining   migrate-tx-partitions-remaining   
1.0.0.127.in-addr.arpa:3000                                 0                                 0   
Number of rows: 1
```


#### Distribution
###### (**Introduced:** 0.1.12)
The `show distribution` displays [histograms](/docs/reference/info#histogram)
and supports the object_size, time_to_live and eviction histograms. But this will 
get displayed if histogram information is provided in collectinfo.

For object_size, we can set parameter `-b` to get bytewise distribution. 

Example Output:
```asciidoc
Collectinfo-analyzer> show distribution time_to_live 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~test - TTL Distribution in Seconds (2016-03-21 12:42:01 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                   Percentage of records having ttl less than or equal to value measured in Seconds                  
                       Node      10%      20%      30%      40%      50%      60%      70%      80%      90%     100%   
1.0.0.127.in-addr.arpa:3000   431900   431900   431900   431900   431900   431900   431900   431900   431900   431900   
1.0.0.127.in-addr.arpa:3100   431900   431900   431900   431900   431900   431900   431900   431900   431900   431900   
1.0.0.127.in-addr.arpa:3200   431900   431900   431900   431900   431900   431900   431900   431900   431900   431900   
Number of rows: 3
```

#### Latency
###### (**Introduced:** 0.1.15)
The `show latency` command displays [latency](/docs/reference/info#latency) characteristics of reads, writes,
and proxies. But this will get displayed if latency information is available in collectinfo.

User can set `-m` to display latency output machine wise. Default display is histogram name wise.

In the below example we look at the latency of writes_master.

```asciidoc
Collectinfo-analyzer> show latency like writes_master
~~~~~~~~~~~~~~~~~~~~~~~~~~~~writes_master Latency~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                    Node                  Time   Ops/Sec  %>1Ms  %>8Ms  %>64Ms
                       .                  Span         .      .      .       .
u10.aerospike.local:3000    23:32:20->23:32:30    2044.7   1.09    0.0     0.0
u12.aerospike.local:3000    23:32:20->23:32:30    2012.6   0.77    0.0     0.0
u13.aerospike.local:3000    23:32:20->23:32:30    1968.9   1.03    0.0     0.0
Number of rows: 3
```

#### Pmap
###### (**Introduced:** 0.1.12)
The `show pmap` command displays partition map analysis of the Aerospike cluster. But this will 
get displayed if partition information is available in collectinfo.

The following example shows the output of the `show pmap` command.

**Primary Partitions:** Total number of primary partitions for a specific namespace on that node.

**Secondary Partitions:** Total number of secondary partitions for a specific namespace on that node.

**Missing Partitions:** The partition count which is neither in "sync state" (S) nor in the "desync state" (D),
for a namespace throughout the cluster are represented as missing partitions.
In a stable cluster the missing partitions should be zero.


```asciidoc
Collectinfo-analyzer> show pmap
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Partition Map Analysis~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
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

Example Output:
```asciidoc
Collectinfo-analyzer> show statistics service like stat_write -t
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Service Statistics (2016-03-21 15:46:40 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                    :   1.0.0.127.in-addr.arpa:3000   1.0.0.127.in-addr.arpa:3100   1.0.0.127.in-addr.arpa:3200   Total   
stat_write_errs         :   0                             0                             0                             0       
stat_write_errs_notfound:   0                             0                             0                             0       
stat_write_errs_other   :   0                             0                             0                             0       
stat_write_reqs         :   666                           652                           682                           2000    
stat_write_reqs_xdr     :   0                             0                             0                             0       
stat_write_success      :   666                           652                           682                           2000
```

```asciidoc
Collectinfo-analyzer> show statistics sets for ns1 set1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ns1 set1 Set Statistics (2018-02-14 13:14:15 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE             :   10.88.66.40:3000  10.88.66.41:3000   10.88.66.42:3000   10.88.66.43:3000   10.88.66.44:3000   
disable-eviction :   false             false              false              false              false              
memory_data_bytes:   0                 0                  0                  0                  0                  
ns               :   ns1               ns1                ns1                ns1                ns1           
objects          :   0                 0                  0                  0                  0                  
set              :   set1              set1               set1               set1               set1       
set-enable-xdr   :   use-default       use-default        use-default        use-default        use-default        
stop-writes-count:   0                 0                  0                  0                  0                  
tombstones       :   0                 0                  0                  0                  0                  
truncate_lut     :   0                 0                  0                  0                  0               
```

For large clusters we can use `-flip` option to flip output table for simplicity and ease of understanding.

Example Output:
```asciidoc
Collectinfo-analyzer> show statistics namespace for test like partition -flip
~~~~~~~~~~~~~~~~~~~~test Namespace Statistics (2016-03-18 14:21:52 UTC)~~~~~~~~~~~~~~~~~~~~
                       NODE   migrate-rx-partitions-initial   migrate-tx-partitions-initial   
1.0.0.127.in-addr.arpa:3000                               0                               0   
Number of rows: 1
```


### Features
The `features` command displays features used in cluster. It supports **like** modifier.

Example Output:
```asciidoc
Collectinfo-analyzer> features
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~(2016-03-22 09:45:56 UTC) Features~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE           :   1.0.0.127.in-addr.arpa:3000   1.0.0.127.in-addr.arpa:3100   1.0.0.127.in-addr.arpa:3200   
AGGREGATION    :   NO                            NO                            NO                            
BATCH          :   NO                            NO                            NO  
INDEX-ON-DEVICE:   NO                            NO                            NO   
INDEX-ON-PMEM  :   NO                            NO                            NO                               
KVS            :   NO                            NO                            YES                           
LDT            :   NO                            NO                            NO                           
QUERY          :   NO                            NO                            NO                            
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
The `summary` command displays summary of cluster. It displays active features used by cluster.

Example Output:
```asciidoc
Collectinfo-analyzer> summary -l
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

### Pager
The `pager` command sets pager for output. For output which can not fit in
output console, this command gives option to scroll each output table vertically as well as horizontally.
