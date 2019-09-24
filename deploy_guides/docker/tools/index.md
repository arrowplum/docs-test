---
title: Deploying Aerospike Tools with Docker
description: |
  Using Docker to deploy and manage your Aeropsike Tools. 
styles:
  - /assets/styles/ui/steps.css

---
## Why Containers?
Containers are seen as a simple way to:

- Encapsulate the dependencies for the process you want to run, e.g. the packages required, the frameworks that need to be present etc. 
- Provides isolation at runtime, enabling containers each with different dependencies, O/S Kernel version etc. to co-exist on the same physical host.

Application architectures are also evolving, especially with the adoption of micro-server like architectures. Your application is no longer one humongous, static binary or set of packages, its starting to look like a series of discrete services that are brought together dynamically at runtime. This is a natural fit for Containers. But how does this fit with services that require persistence or require long running processes?

## Getting started

### Install Docker 
Docker supports multiple platforms on Desktop(Microsoft Windows 10 & macOS), Cloud(Amazon Web Service & Microsoft Azure), and Server(CentOS, Debian, Fedora, Ubuntu).

Listings with installation links can be found [here](https://docs.docker.com/install/#supported-platforms)

#### Example: Install Docker for Mac
As an example we are providing instructions for installing on a Mac environment.

The instructions for installing Docker for Mac can be found [here](https://docs.docker.com/docker-for-mac/install/).

The download for Docker Community Edition for Mac is available for free located [here](https://store.docker.com/editions/community/docker-ce-desktop-mac).

Docker CE for Mac is an easy-to-install desktop app for building, debugging, and testing Dockerized apps on a Mac. Docker for Mac is a complete development environment deeply integrated with the Mac OS Hypervisor framework, networking, and filesystem. Docker for Mac is the fastest and most reliable way to run Docker on a Mac.

#### Example: Launch Docker for Mac

As an example we are providing instructions for launching docker on a Mac environment.

First, launch the Docker Application

Then in a Terminal shell verify docker is running using the following command:
```bash
$ docker --version
Docker version 18.03.1-ce, build 9ee9f40
```
 
### Available tools:
Aerospike provides [prepackaged tools images](https://hub.docker.com/r/aerospike/aerospike-tools/~/dockerfile/) for Docker with the following Aerospike Tools:

   * asadm 
   * asinfo 
   * aql 
   * asloglatency 
   * asbackup 
   * asrestore 

Additional info on using our tools can be found [here](/docs/tools/).

## Usage

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]</code>
```

```bash
docker run -ti -v <host>:<container> --name aerospike-tools --rm aerospike/aerospike-tools <Aerospike Tools commands> -h <Seed_Host_IP> --no-config-file
```
| Option            | Description                          |
|-------------------|--------------------------------------|
| -t, --tty         | Allocate a pseudo-TTY                |
| -i, --interactive | Keep STDIN open even if not attached |
| -v, --volume list | Bind mount a volume                  |
| --name            | Container name                       |
| --rm              | Remove container after use           |

### Aerospike Admin Tool (asadm)

The following will run the [Aerospike Admin Tool (asadm)](/docs/tools/asadm/index.html).

```bash
$ docker run -ti --name aerospike-asadm —-rm aerospike/aerospike-tools asadm <Aerospike Admin Tool commands> --host <Seed_Host_IP> --no-config-file
```

#### Examples

To run asadm allowing for asadm command line input use the following:

```bash
$ docker run -ti --name aerospike-asadm --rm aerospike/aerospike-tools asadm --host 10.0.0.173 --no-config-file
Seed:        [('10.0.0.173', 3000, None)]
Config_file: None
Aerospike Interactive Shell, version 0.1.18

Found 2 nodes
Online:  10.0.0.171:3000, 10.0.0.173:3000

Admin> 
```

You can inspect the Aerospike Cluster configuration using:

```bash
$ docker run -ti --name aerospike-asadm --rm aerospike/aerospike-tools asadm -e info --host 10.0.0.173 --no-config-file 
Seed:        [('10.0.0.173', 3000, None)]
Config_file: None
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information (2018-05-10 08:47:57 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
           Node               Node                Ip        Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime   
              .                 Id                 .            .      Size            .            Key   Integrity                 .    Conns          .   
10.0.0.171:3000   *BB9BC6479270008   10.0.0.171:3000   E-3.15.0.2         2      0.000     6F420A0B3DA7   True        BB9BC6479270008        1   00:16:45   
10.0.0.173:3000   BB945718C270008    10.0.0.173:3000   E-4.1.0.1          2      0.000     6F420A0B3DA7   True        BB9BC6479270008        4   00:14:27   
Number of rows: 2

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Usage Information (2018-05-10 08:47:57 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace              Node     Total   Expirations,Evictions     Stop       Disk    Disk     HWM   Avail%        Mem     Mem    HWM      Stop   
        .                 .   Records                       .   Writes       Used   Used%   Disk%        .       Used   Used%   Mem%   Writes%   
bar         10.0.0.171:3000   0.000     (0.000,  0.000)         false         N/E   N/E     50      N/E      0.000 B    0       60     90        
bar         10.0.0.173:3000   0.000     (0.000,  0.000)         false         N/E   N/E     50      N/E      0.000 B    0       60     90        
bar                           0.000     (0.000,  0.000)                  0.000 B                             0.000 B                             
test        10.0.0.171:3000   0.000     (0.000,  0.000)         false         N/E   N/E     50      N/E      0.000 B    0       60     90        
test        10.0.0.173:3000   0.000     (0.000,  0.000)         false         N/E   N/E     50      N/E      0.000 B    0       60     90        
test                          0.000     (0.000,  0.000)                  0.000 B                             0.000 B                             
Number of rows: 6

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Object Information (2018-05-10 08:47:57 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace              Node     Total     Repl                      Objects                   Tombstones             Pending   Rack   
        .                 .   Records   Factor   (Master,Prole,Non-Replica)   (Master,Prole,Non-Replica)            Migrates     ID   
        .                 .         .        .                            .                            .             (tx,rx)      .   
bar         10.0.0.171:3000   0.000     2        (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)     0      
bar         10.0.0.173:3000   0.000     2        (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)     0      
bar                           0.000              (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)            
test        10.0.0.171:3000   0.000     2        (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)     0      
test        10.0.0.173:3000   0.000     2        (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)     0      
test                          0.000              (0.000,  0.000,  0.000)      (0.000,  0.000,  0.000)      (0.000,  0.000)            
Number of rows: 6
```

You can issue Aerospike Information Tools (asinfo) commands, in this case returning the build version for each node on all cluster nodes at once using:

```bash
$ docker run -ti --name aerospike-asadm --rm aerospike/aerospike-tools asadm -e 'asinfo -v build' --host 10.0.0.173 --no-config-file
Seed:        [('10.0.0.173', 3000, None)]
Config_file: None
10.0.0.171:3000 (10.0.0.171) returned:
3.15.0.2

10.0.0.173:3000 (10.0.0.173) returned:
4.1.0.1
```

### Aerospike Information Tool (asinfo)

The following will run the [Aerospike Information Tool (asinfo)](/docs/tools/asinfo).

```bash
$ docker run -ti --name aerospike-asinfo —-rm aerospike/aerospike-tools asinfo <Aerospike Information Tool commands> --host <Seed_Host_IP> --no-config-file
```

#### Examples

To run asinfo displaying server configuration parameters use the following:

```bash
$ docker run -ti --name aerospike-asinfo --rm aerospike/aerospike-tools asinfo -h 10.0.0.173 --no-config-file
1 :  node
     BB945718C270008
2 :  statistics
     cluster_size=2;cluster_key=BD70190EBEC1;cluster_generation=1;cluster_principal=BB945718C270008;cluster_integrity=true;cluster_is_member=true;cluster_duplicate_nodes=null;cluster_clock_skew_stop_writes_sec=56;cluster_clock_skew_ms=0;cluster_clock_skew_outliers=null;uptime=21500;system_free_mem_pct=88;heap_allocated_kbytes=171468;heap_active_kbytes=173940;heap_mapped_kbytes=322560;heap_efficiency_pct=53;heap_site_count=41;objects=0;tombstones=0;tsvc_queue=0;info_queue=0;rw_in_progress=0;proxy_in_progress=0;tree_gc_queue=0;client_connections=1;heartbeat_connections=0;fabric_connections=24;heartbeat_received_self=42987;heartbeat_received_foreign=42986;reaped_fds=7;info_complete=5120;demarshal_error=0;early_tsvc_client_error=0;early_tsvc_from_proxy_error=0;early_tsvc_batch_sub_error=0;early_tsvc_from_proxy_batch_sub_error=0;early_tsvc_udf_sub_error=0;batch_index_initiate=0;batch_index_queue=0:0,0:0;batch_index_complete=0;batch_index_error=0;batch_index_timeout=0;batch_index_delay=0;batch_index_unused_buffers=0;batch_index_huge_buffers=0;batch_index_created_buffers=0;batch_index_destroyed_buffers=0;scans_active=0;query_short_running=0;query_long_running=0;sindex_ucgarbage_found=0;sindex_gc_retries=0;sindex_gc_list_creation_time=861;sindex_gc_list_deletion_time=0;sindex_gc_objects_validated=514;sindex_gc_garbage_found=0;sindex_gc_garbage_cleaned=0;paxos_principal=BB945718C270008;time_since_rebalance=21494;migrate_allowed=true;migrate_partitions_remaining=0;fabric_bulk_send_rate=0;fabric_bulk_recv_rate=0;fabric_ctrl_send_rate=0;fabric_ctrl_recv_rate=0;fabric_meta_send_rate=298;fabric_meta_recv_rate=298;fabric_rw_send_rate=0;fabric_rw_recv_rate=0;dlog_used_objects=61;dlog_free_pct=100;dlog_logged=30761;dlog_relogged=5508;dlog_processed_main=30761;dlog_processed_replica=145;dlog_processed_link_down=0;dlog_overwritten_error=0;xdr_ship_success=0;xdr_ship_delete_success=0;xdr_ship_source_error=5508;xdr_ship_destination_error=0;xdr_ship_destination_permanent_error=0;xdr_ship_fullrecord=5508;xdr_ship_bytes=0;xdr_ship_inflight_objects=0;xdr_ship_outstanding_objects=0;xdr_ship_latency_avg=0;xdr_ship_compression_avg_pct=0.00;xdr_read_success=5508;xdr_read_error=0;xdr_read_notfound=528;xdr_read_latency_avg=0;xdr_read_active_avg_pct=0.00;xdr_read_idle_avg_pct=100.00;xdr_read_reqq_used=0;xdr_read_respq_used=0;xdr_read_reqq_used_pct=0.00;xdr_read_txnq_used=0;xdr_read_txnq_used_pct=0.00;xdr_relogged_incoming=0;xdr_relogged_outgoing=5508;xdr_queue_overflow_error=0;xdr_active_failed_node_sessions=0;xdr_active_link_down_sessions=0;xdr_hotkey_fetch=1638;xdr_hotkey_skip=26363;xdr_unknown_namespace_error=0;xdr_timelag=0;xdr_throughput=0;xdr_global_lastshiptime=1551340161340
3 :  features
     batch-index;cdt-list;cdt-map;cluster-stable;float;geo;peers;pipelining;replicas;replicas-all;replicas-master;truncate-namespace;udf;xdr
4 :  partition-generation
     8192
5 :  build_time
     Mon Feb 25 22:48:10 UTC 2019
6 :  dcs
     DC2;DC3;DC4;DC5;DC6;DC7
7 :  edition
     Aerospike Enterprise Edition
8 :  version
     Aerospike Enterprise Edition build 4.5.1.5
9 :  compatibility-id
     1
10 :  services
     10.0.0.171:3000
11 :  services-alumni
     10.0.0.171:3000
12 :  build_os
     debian9 
13 :  build
     4.5.1.5
```

To run asinfo displaying all namespaces use the following:

```bash
$ docker run -ti --name aerospike-asinfo --rm aerospike/aerospike-tools asinfo -v "namespaces" -h 10.0.0.173 --no-config-file
test;bar
```

To run asinfo displaying all statistics use the following:

```bash
$ docker run -ti --name aerospike-asinfo --rm aerospike/aerospike-tools asinfo -v statistics -h 10.0.0.173 --no-config-file
cluster_size=2;cluster_key=BD70190EBEC1;cluster_generation=1;cluster_principal=BB945718C270008;cluster_integrity=true;cluster_is_member=true;cluster_duplicate_nodes=null;cluster_clock_skew_stop_writes_sec=56;cluster_clock_skew_ms=0;cluster_clock_skew_outliers=null;uptime=21500;system_free_mem_pct=88;heap_allocated_kbytes=171468;heap_active_kbytes=173940;heap_mapped_kbytes=322560;heap_efficiency_pct=53;heap_site_count=41;objects=0;tombstones=0;tsvc_queue=0;info_queue=0;rw_in_progress=0;proxy_in_progress=0;tree_gc_queue=0;client_connections=1;heartbeat_connections=0;fabric_connections=24;heartbeat_received_self=42987;heartbeat_received_foreign=42986;reaped_fds=7;info_complete=5120;demarshal_error=0;early_tsvc_client_error=0;early_tsvc_from_proxy_error=0;early_tsvc_batch_sub_error=0;early_tsvc_from_proxy_batch_sub_error=0;early_tsvc_udf_sub_error=0;batch_index_initiate=0;batch_index_queue=0:0,0:0;batch_index_complete=0;batch_index_error=0;batch_index_timeout=0;batch_index_delay=0;batch_index_unused_buffers=0;batch_index_huge_buffers=0;batch_index_created_buffers=0;batch_index_destroyed_buffers=0;scans_active=0;query_short_running=0;query_long_running=0;sindex_ucgarbage_found=0;sindex_gc_retries=0;sindex_gc_list_creation_time=861;sindex_gc_list_deletion_time=0;sindex_gc_objects_validated=514;sindex_gc_garbage_found=0;sindex_gc_garbage_cleaned=0;paxos_principal=BB945718C270008;time_since_rebalance=21494;migrate_allowed=true;migrate_partitions_remaining=0;fabric_bulk_send_rate=0;fabric_bulk_recv_rate=0;fabric_ctrl_send_rate=0;fabric_ctrl_recv_rate=0;fabric_meta_send_rate=298;fabric_meta_recv_rate=298;fabric_rw_send_rate=0;fabric_rw_recv_rate=0;dlog_used_objects=61;dlog_free_pct=100;dlog_logged=30761;dlog_relogged=5508;dlog_processed_main=30761;dlog_processed_replica=145;dlog_processed_link_down=0;dlog_overwritten_error=0;xdr_ship_success=0;xdr_ship_delete_success=0;xdr_ship_source_error=5508;xdr_ship_destination_error=0;xdr_ship_destination_permanent_error=0;xdr_ship_fullrecord=5508;xdr_ship_bytes=0;xdr_ship_inflight_objects=0;xdr_ship_outstanding_objects=0;xdr_ship_latency_avg=0;xdr_ship_compression_avg_pct=0.00;xdr_read_success=5508;xdr_read_error=0;xdr_read_notfound=528;xdr_read_latency_avg=0;xdr_read_active_avg_pct=0.00;xdr_read_idle_avg_pct=100.00;xdr_read_reqq_used=0;xdr_read_respq_used=0;xdr_read_reqq_used_pct=0.00;xdr_read_txnq_used=0;xdr_read_txnq_used_pct=0.00;xdr_relogged_incoming=0;xdr_relogged_outgoing=5508;xdr_queue_overflow_error=0;xdr_active_failed_node_sessions=0;xdr_active_link_down_sessions=0;xdr_hotkey_fetch=1638;xdr_hotkey_skip=26363;xdr_unknown_namespace_error=0;xdr_timelag=0;xdr_throughput=0;xdr_global_lastshiptime=1551340161340
```

To run asinfo displaying all features replacing semicolons ';' in with line breaks in the response use the following:

```bash
$ docker run -ti --name aerospike-asinfo --rm aerospike/aerospike-tools asinfo -v features -l -h 10.0.0.173 --no-config-file
peers
cdt-list
cdt-map
pipelining
geo
float
batch-index
replicas
replicas-all
replicas-master
replicas-prole
udf
xdr
```

To get the configuration value for the service context proto-fd-max run the following asinfo -v get-config command piping the output to grep for the specific configuration value: 

```bash
$ docker run -ti --name aerospike-asinfo --rm aerospike/aerospike-tools asinfo -v "get-config:context=service" -l -h 10.0.0.173 --no-config-file | grep proto-fd-max
proto-fd-max=15000
```

To set the configuration value for the service context proto-fd-max run the following asinfo -v set-config command:
```bash
$ docker run -ti --name aerospike-asinfo --rm aerospike/aerospike-tools asinfo -v 'set-config:context=service;proto-fd-max=100000' -h 10.0.0.173 --no-config-file
ok
```

### AQL Tool (aql)

The following will run the [AQL Tool (aql)](/docs/tools/aql/index.html).

```bash
$ docker run -ti --name aerospike-aql —-rm aerospike/aerospike-tools aql <AQL Tool commands> --host <Seed_Host_IP> --no-config-file
```

#### Examples

To allow for aql command line input use the following:

```bash
$ docker run -ti --name aerospike-aql --rm aerospike/aerospike-tools aql -h 10.0.0.173 --no-config-file
Seed:         10.0.0.173
User:         None
Config File:  None
Aerospike Query Client
Version 3.15.3.6
C Client Version 4.3.11
Copyright 2012-2017 Aerospike. All rights reserved.
aql> 
```

You can run [AQL](/docs/tools/aql) in a Docker container to insert and query some data into the Aerospike server.

```bash
$ docker run -ti --name aerospike-aql --rm aerospike/aerospike-tools aql --host 10.0.0.173 --no-config-file
Seed:         10.0.0.173
User:         None
Config File:  None
Aerospike Query Client
Version 3.15.3.6
C Client Version 4.3.11
Copyright 2012-2017 Aerospike. All rights reserved.
aql> insert into test.foo (PK, foo) values ('123','my string')
OK, 1 record affected.

aql> select * from test.foo
+-------------+
| foo         |
+-------------+
| "my string" |
+-------------+
1 row in set (0.126 secs)

OK

aql>
```

To execute AQL directly inserting a record run the following command.

```bash
$ docker run -ti --name aerospike-aql --rm aerospike/aerospike-tools aql -c "INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', 123, 'abc')" -h 10.0.0.173 --no-config-file
INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', 123, 'abc')
OK, 1 record affected.
```

To execute AQL directly returning the details for a single record run the following command:

```bash
$ docker run -ti --name aerospike-aql --rm aerospike/aerospike-tools aql -c "explain select * from test.foo where PK='123'" -h 10.0.0.173 --no-config-file
explain select * from test.foo where PK='123'
+-------+--------------------------------------------+-----------+-----------+--------+---------+----------+----------------------------+-------------------+-------------------------+---------+
| SET   | DIGEST                                     | NAMESPACE | PARTITION | STATUS | UDF     | KEY_TYPE | POLICY_REPLICA             | NODE              | POLICY_KEY              | TIMEOUT |
+-------+--------------------------------------------+-----------+-----------+--------+---------+----------+----------------------------+-------------------+-------------------------+---------+
| "foo" | "44E3571220664C352DFCC7EFD681920D37F414AC" | "test"    | 836       | ""     | "FALSE" | "STRING" | "AS_POLICY_REPLICA_MASTER" | "BB945718C270008" | "AS_POLICY_KEY_DEFAULT" | 1000    |
+-------+--------------------------------------------+-----------+-----------+--------+---------+----------+----------------------------+-------------------+-------------------------+---------+
1 row in set (0.001 secs)

OK
```

To execute AQL directly truncating a set(demo) in a namespace(test) run the following command:

```bash
$ docker run -ti --name aerospike-aql --rm aerospike/aerospike-tools aql -c "TRUNCATE test.demo" -h 10.0.0.173 --no-config-file
TRUNCATE test.demo
OK
```

### Aerospike Log Latency Tool (asloglatency)

The following will run the [Aerospike Log Latency Tool (asloglatency)](/docs/tools/asloglatency/index.html).

```bash
$ docker run -ti -v host:container --name aerospike-asloglatency —-rm aerospike/aerospike-tools asloglatency <Aerospike Log Latency Tool commands>
```

#### Example

To run asloglatency against an aerospike.log file located on the local machine /tmp directory querying the read histogram analyzing the Log time from the beginning of the file against the namespace test displaying 10 buckets showing the 0-th and then every 1-th bucket that is located on the bind mounted volume of the present working directory subdirectory tmp as /tmp run the following command:

```bash
$ docker run -ti -v ${PWD}/tmp:/tmp --name aerospike-asloglatency --rm aerospike/aerospike-tools asloglatency -l /tmp/aerospike.log -h read -f head -N test -n 10 -e 1
Histogram : {test}-read
Log       : /tmp/aerospike.log
From      : 2018-05-09 00:20:02

May 09 2018 00:20:02
               % > (ms)
slice-to (sec)      1      2      4      8     16     32     64    128    256    512    ops/sec
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ----------
00:20:12    10   0.29   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      103.6
00:20:22    10   0.26   0.09   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      113.5
00:20:32    10   0.19   0.09   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      105.9
00:20:42    10   0.25   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      118.4
00:20:52    10   0.29   0.19   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      105.0
00:21:02    10   0.17   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      120.3
00:21:12    10   0.36   0.27   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      111.0
00:21:22    10   0.44   0.18   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      112.4
00:21:32    10   0.45   0.09   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      111.0
00:21:42    10   0.46   0.09   0.09   0.00   0.00   0.00   0.00   0.00   0.00   0.00      109.5
00:21:52    10   0.44   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00   0.00      114.4
```

### Aerospike Backup Tool (asbackup)

The following will run the [Aerospike Backup Tool(asbackup)](/docs/tools/backup/asbackup.html).

```bash
$ docker run -ti -v host:container --name aerospike-asbackup —-rm aerospike/aerospike-tools asbackup <Aerospike Backup Tool commands> --host <Seed_Host_IP> --no-config-file
```

#### Examples

To execute asbackup generating a backup that is located on the bind mounted volume of the present working directory subdirectory asbackup  as /tmp/asbackup, clearing the directory /tmp/asbackup,  run the following command: 

```bash
$ docker run -ti -v ${PWD}/asbackup:/tmp/asbackup --name aerospike-asbackup --rm aerospike/aerospike-tools asbackup --namespace test --directory /tmp/asbackup -r --host 10.0.0.173 --no-config-file
2018-06-21 02:18:14 GMT [INF] [    8] Starting 100% backup of 10.0.0.173 (namespace: test, set: [all], bins: [all], after: [none], before: [none]) to /tmp/asbackup
2018-06-21 02:18:14 GMT [INF] [    8] [src/main/aerospike/as_cluster.c:124][as_cluster_add_nodes_copy] Add node BB945718C270008 10.0.0.173:3000
2018-06-21 02:18:14 GMT [INF] [    8] [src/main/aerospike/as_cluster.c:124][as_cluster_add_nodes_copy] Add node BB9BC6479270008 10.0.0.171:3000
2018-06-21 02:18:14 GMT [INF] [    8] Processing 2 node(s)
2018-06-21 02:18:14 GMT [INF] [    8] Node ID             Objects        Replication    
2018-06-21 02:18:14 GMT [INF] [    8] BB945718C270008     2              2              
2018-06-21 02:18:14 GMT [INF] [    8] BB9BC6479270008     2              2              
2018-06-21 02:18:14 GMT [INF] [    8] Namespace contains 2 record(s)
2018-06-21 02:18:14 GMT [INF] [    8] Directory /tmp/asbackup prepared for backup
2018-06-21 02:18:14 GMT [INF] [   27] Starting backup for node BB9BC6479270008
2018-06-21 02:18:14 GMT [INF] [   28] Starting backup for node BB945718C270008
2018-06-21 02:18:14 GMT [INF] [   27] Created new backup file /tmp/asbackup/BB9BC6479270008_00000.asb
2018-06-21 02:18:14 GMT [INF] [   28] Created new backup file /tmp/asbackup/BB945718C270008_00000.asb
2018-06-21 02:18:14 GMT [INF] [   28] Backing up 1 secondary index(es)
2018-06-21 02:18:14 GMT [INF] [   28] Backing up 1 UDF file(s)
2018-06-21 02:18:14 GMT [INF] [   27] Completed backup for node BB9BC6479270008, records: 1, size: 137 (~137 B/rec)
2018-06-21 02:18:14 GMT [INF] [   28] Completed backup for node BB945718C270008, records: 1, size: 244 (~244 B/rec)
2018-06-21 02:18:15 GMT [INF] [   26] Backed up 2 record(s), 1 secondary index(es), 1 UDF file(s) from 2 node(s), 381 byte(s) in total (~190 B/rec)
```

To allow for pipelines, specify - as the `--output-file` and asbackup writes the backup to stdout and also specify - as the `--input-file` and asrestore reads the backup from stdin.

```bash
$ docker run -i --name aerospike-asbackup --rm  aerospike/aerospike-tools asbackup --namespace test --output-file - --host 10.0.0.173 --no-config-file | docker run -i --name aerospike-asrestore --rm aerospike/aerospike-tools asrestore --namespace test -g --host 10.0.0.173 --no-config-file --input-file -
2018-06-22 03:19:02 GMT [INF] [    8] Starting 100% backup of 10.0.0.173 (namespace: test, set: [all], bins: [all], after: [none], before: [none]) to [stdout]
2018-06-22 03:19:02 GMT [INF] [    8] Starting restore to 10.0.0.173 (bins: [all], sets: [all]) from [stdin]
2018-06-22 03:19:02 GMT [INF] [    8] [src/main/aerospike/as_cluster.c:124][as_cluster_add_nodes_copy] Add node BB945718C270008 10.0.0.173:3000
2018-06-22 03:19:03 GMT [INF] [    8] [src/main/aerospike/as_cluster.c:124][as_cluster_add_nodes_copy] Add node BB9BC6479270008 10.0.0.171:3000
2018-06-22 03:19:03 GMT [INF] [    8] Processing 2 node(s)
2018-06-22 03:19:03 GMT [INF] [    8] Node ID             Objects        Replication    
2018-06-22 03:19:03 GMT [INF] [    8] Processing 2 node(s)
2018-06-22 03:19:03 GMT [INF] [    8] Restoring -
2018-06-22 03:19:03 GMT [INF] [    8] BB945718C270008     5              2              
2018-06-22 03:19:03 GMT [INF] [    8] BB9BC6479270008     5              2              
2018-06-22 03:19:03 GMT [INF] [    8] Namespace contains 5 record(s)
2018-06-22 03:19:03 GMT [INF] [   27] Starting backup for node BB945718C270008
2018-06-22 03:19:03 GMT [INF] [   28] Starting backup for node BB9BC6479270008
2018-06-22 03:19:03 GMT [INF] [   27] Backing up 1 secondary index(es)
2018-06-22 03:19:03 GMT [INF] [   27] Backing up 1 UDF file(s)
2018-06-22 03:19:03 GMT [INF] [   27] Completed backup for node BB945718C270008, records: 2, size: 349 (~174 B/rec)
2018-06-22 03:19:03 GMT [INF] [   28] Completed backup for node BB9BC6479270008, records: 3, size: 315 (~105 B/rec)
2018-06-22 03:19:03 GMT [INF] [    8] Restoring 1 UDF file(s)
2018-06-22 03:19:03 GMT [INF] [    8] Restoring 1 secondary index(es)
2018-06-22 03:19:03 GMT [INF] [    8] Skipped 1 matched index(es)
2018-06-22 03:19:03 GMT [INF] [    8] Restoring records
2018-06-22 03:19:04 GMT [INF] [   26] 1 UDF file(s), 0 secondary index(es), 5 record(s) (0 KiB/s, 5 rec/s, 132 B/rec, backed off: 0)
2018-06-22 03:19:04 GMT [INF] [   26] Expired 0 : skipped 0 : inserted 5 : failed 0 (existed 0, fresher 0)
2018-06-22 03:19:04 GMT [INF] [   26] Backed up 5 record(s), 1 secondary index(es), 1 UDF file(s) from 2 node(s), 664 byte(s) in total (~132 B/rec)
2018-06-22 03:19:05 GMT [INF] [   26] 1 UDF file(s), 0 secondary index(es), 5 record(s) (0 KiB/s, 0 rec/s, 0 B/rec, backed off: 0)
2018-06-22 03:19:05 GMT [INF] [   26] Expired 0 : skipped 0 : inserted 5 : failed 0 (existed 0, fresher 0)
```

### Aerospike Restore Tool (asrestore)

The following will run the [Aerospike Restore Tool(asrestore)](/docs/tools/backup/asrestore.html).

```bash
$ docker run -ti -v host:container --name aerospike-asrestore —-rm aerospike/aerospike-tools asrestore <Aerospike Restore Tool commands> --host <Seed_Host_IP> --no-config-file
```

#### Example
 
To execute asrestore restoring from a backup that is located on the bind mounted volume of the present working directory subdirectory asbackup as /tmp/asbackup, the backup always overwrite records that already exist in the namespace, run the following command:

```bash
$ docker run -ti -v ${PWD}/asbackup:/tmp/asbackup --name aerospike-asrestore --rm aerospike/aerospike-tools asrestore --namespace test --directory /tmp/asbackup -g --host 10.0.0.173 --no-config-file
2018-06-21 02:19:25 GMT [INF] [    8] Starting restore to 10.0.0.173 (bins: [all], sets: [all]) from /tmp/asbackup
2018-06-21 02:19:26 GMT [INF] [    8] Processing 2 node(s)
2018-06-21 02:19:26 GMT [INF] [    8] Found 2 backup file(s) in /tmp/asbackup
2018-06-21 02:19:26 GMT [INF] [    8] Opened backup file /tmp/asbackup/BB945718C270008_00000.asb
2018-06-21 02:19:26 GMT [INF] [    8] Opened backup file /tmp/asbackup/BB9BC6479270008_00000.asb
2018-06-21 02:19:26 GMT [INF] [    8] Restoring 1 UDF file(s)
2018-06-21 02:19:26 GMT [INF] [    8] Restoring 1 secondary index(es)
2018-06-21 02:19:26 GMT [INF] [    8] Skipped 1 matched index(es)
2018-06-21 02:19:26 GMT [INF] [    8] Restoring records
2018-06-21 02:19:26 GMT [INF] [   27] Restoring /tmp/asbackup/BB945718C270008_00000.asb
2018-06-21 02:19:26 GMT [INF] [   28] Restoring /tmp/asbackup/BB9BC6479270008_00000.asb
2018-06-21 02:19:26 GMT [INF] [   28] Opened backup file /tmp/asbackup/BB9BC6479270008_00000.asb
2018-06-21 02:19:26 GMT [INF] [   27] Opened backup file /tmp/asbackup/BB945718C270008_00000.asb
2018-06-21 02:19:27 GMT [INF] [   26] 1 UDF file(s), 0 secondary index(es), 2 record(s) (0 KiB/s, 2 rec/s, 190 B/rec, backed off: 0)
2018-06-21 02:19:27 GMT [INF] [   26] Expired 0 : skipped 0 : inserted 2 : failed 0 (existed 0, fresher 0)
2018-06-21 02:19:27 GMT [INF] [   26] 100% complete, ~0s remaining
```

## Source Code
The source code is available on <strong><a id="github" href="https://github.com/aerospike/aerospike-tools.docker">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a></strong>

