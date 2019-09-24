---
title: Advanced Flash Device Sizing
description: Learn how to test Flash devices for a specific production workload

---
    
Since its inception in 2012 the Aerospike Certification Tool ([ACT](https://github.com/aerospike/act)) has grown to become an industry standard Flash 
drive test tool for operational database use cases. ACT generates a real world workload on Flash drives like a workload 
that occurs in an Aerospike database.

While Aerospike does not require any particular Flash device, and can be run on SAN devices, network storage, file systems,
and even a pure in-memory mode, performance of the storage system is key to a predictable database. Aerospike's
workload is different from other databases, and storage metrics used for other databases (such as pure IOP or 4K 
read and write performance) are not a good predictor of performance.

Recent improvements enable ACT to fine tune the workloads on drives to deliver a closer representation of an actual 
workload created by Aerospike. The Aerospike server has parameters that expose the object sizes in use, 
the effectiveness of internal cache and defragmentation processes.

ACT originally allowed specification of a database load simply: reads per second, writes per second and object size. 
From this, the device's workload would be simulated, with a default replication factor. 
However, there are now additional factors in Aerospike , such as read-modify-write, other replication factors, 
database "tombstone" for [durable deletes](/docs/guide/durable_deletes.html), and Aerospike's new [`commit-to-device`](/docs/reference/configuration/#commit-to-device) mode. 

Previously, we considered a drive certified if it could run with 1.5 KB objects in a 50/50 read/write workload with less than 5% of responses 
taking more than one millisecond. However, with Aerospike used in a broad range of uses, we find that original profile
to be outdated. Some use cases are similar, but some have larger and smaller objects, different ratios, and different
latency requirements.

The latest ACT 4.0 changes provide adjustments to model these extra loads. ACT also has a new option to define a 
range of object sizes rather than just a single object size, which can induce different behaviors in drives. 
The release of ACT 4.0 provides a tool that is more flexible in creating workloads on a drive closer to a production workload.  
 
The Aerospike cluster can also now emit statistics which can be run directly into ACT, thus allowing easier and far 
more accurate ACT profiles if you have a running production system. After capturing data from a production cluster, 
the data can be entered into the ACT configuration file. ACT will create a range of object sizes based on those seen 
in the production cluster and generate a workload on the drive similar to the that seen on the cluster. This mode of 
operation will allow existing users of Aerospike to test and select drives to upgrade their clusters with a better 
confidence level that the drives meet their SLAs.
 
ACT can be used with or without cluster data to produce the data needed for drive selection. If you don’t 
currently use Aerospike, the latest options and capabilities will provide the results you need to select a drive. 

The rest of this paper describes how to take histogram data from an Aerospike cluster and using it to 
configure ACT for testing and drive selection.
 
This paper can be read in whole or in part. Depending on your level of experience with Aerospike and ACT you may 
want to read all or some of the sections. The following briefly describes the contents of each section.  
  

**Prerequisites for the process** - These are the systems and tools needed for creating the same process in your own environment.  

**Setting up and environment** - A simple process of installing the tools needed for the process. If you are already familiar with ACT and Aerospike you may scan or skip this section.

**Collecting cluster performance** - This is an important section. It is the process of collecting the data that will be used in the testing process.  Most of the data will come from the production cluster that is being considered for hardware upgrade.

**Configuring ACT** - This section explains the new configuration items and how to modify the ACT configuration file for testing.

**Running ACT** - If you are familiar with ACT, this short section can be skipped.  

**Analyzing results and making adjustments** - It is necessary to review the preliminary parameters and match them 
to the operational system. Based on the analysis, various recommendations are made for adjustments to the ACT configuration file.

**Testing new drives** - After the ACT configuration is created and has been validated, the new drives must be evaluated. 
This section describes a strategy for benchmarking the performance and selecting the new drive.

**Final thoughts** - With all the performance testing completed, there are other factors that may be relevant in 
your testing and selection process.  This section reviews some of the most common factors other than performance that should be considered.

**Appendices** (including the environment for this example) - This section briefly describes the environment and tool 
used for generating the data and information. It provides a background for the systems used throughout this process. 

## Prerequisites for the process
 
Before getting into the process, here are a few things that are needed.  

Server:  This server will be the system for running the ACT tests.  This server should preferably be like the servers in production, but will likely have far less DRAM and while it may have less CPU resources, the more similar the better.
    
Production Flash:  One Flash that is the same model of Flash that are used on the production servers, although validating a test with the same multiple of Flash drives is preferred.

Access to a production cluster to collect data.

New SSDs:   Drives that are possible candidates for an upgrade.
    
Tools:  Aerospike [ACT](https://github.com/aerospike/act), [asloglatency](/docs/tools/asloglatency/index.html), and [asinfo](/docs/tools/asinfo/index.html), as well as standard Linux iostat and perf-tools.

## Setting up an environment

### Setting up your ACT test server

Install iostat(Centos install example).  The sysstat package that contains iostat needs to be installed.
```
yum install sysstat  
```

### Install ACT

Download from Github 

```
git clone https://github.com/aerospike/act.git
```

### Build ACT

`cd` to the ACT directory.

```
make
make -f Makesalt
```

Install perf-tools

Clone from Github

Check to see if 'iosnoop' and 'iolatency' are already on the test machine

```
git clone https://github.com/brendangregg/perf-tools.git
```

Add the perf-tools path to the environment PATH.

Validate 'iosnoop' and 'iolatency' work.

Install and configure the drive from the production cluster. 

Configure over provisioning for the drive if it used on the cluster.

https://www.aerospike.com/docs/operations/plan/ssd/ssd_setup.html

Configure controller optimizations if they apply.

https://www.aerospike.com/docs/operations/plan/ssd/lsi_megacli.html

Generate a basic ACT configuration file.  This is the configuration file that will be manually modified throughout the process.

cd to the ACT directory.

Run the ACT helper script.

```
python act_config_helper.py
Enter the number of devices you want to create config for: 1
Enter either raw device if over-provisioned using hdparm or partition if over-provisioned using fdisk:
Enter device name #1 (e.g. /dev/sdb or /dev/sdb1): /dev/nvme0n1
Change test duration default of 24 hours? (y/N): 
Use non-standard configuration? (y/N): 
"1x" load is  2000 reads per second and 1000 writes per second.
Enter the load factor (e.g. enter 1 for 1x test): 
"1x" load is  2000 reads per second and 1000 writes per second.
Enter the load factor (e.g. enter 1 for 1x test): 3
Do you want to save this config to a file? (y/N): y
Config file actconfig_3x_1d.txt successfully created.
```

The following is a sample of the file that will be generated.

```
##########
act config file for testing 1 device(s) at 3x load
##########
 
# comma-separated list:
device-names: /dev/nvme0n1
 
# mandatory non-zero:
num-queues: 8
threads-per-queue: 8
test-duration-sec: 86400
report-interval-sec: 1
large-block-op-kbytes: 128
 
record-bytes: 1536
read-reqs-per-sec: 6000
 
# usually non-zero:
write-reqs-per-sec: 3000
# yes|no - default is no:
microsecond-histograms: no
 
# noop|cfq - default is noop:
scheduler-mode: noop
```

### Collecting Cluster Data

Before using the commands for collecting data on a production cluster make sure they are safe for your environment.  Try the 
commands on a development cluster if it is available.  The commands have been tried in the labs here at Aerospike 
without adverse effects on the performance of the cluster.  After setting up the server you will need to collect some 
data from the production cluster during a steady state peak load period. 

First enable the storage histograms on the cluster.
```
asinfo -v 'set-config:context=namespace;id=ycsb;enable-benchmarks-storage=true'
```

Specify the namespace that you will be enabling for the histogram.

If the drive is shared across multiple namespaces then enable the histogram for all of them.
Logging Detail will also need to be enabled with asinfo. Be careful not to enable detail for all contexts.  This example only enables the detail for the drv_ssd context.
```
asinfo -v "log-set:id=0;drv_ssd=detail"
```

Using asloglatency collect the read histogram latencies for the drive from the namespaces that contain the drive being measured. 
In this example  the drive is being used by only one namespace(ycsb).  It is possible the drive has multiple partitions and each partition is used by a different namespace.

```
[root@Dell6]# asloglatency -f -3000  -t 500 -n 8 -e 1 -h {ycsb}-read -l /var/log/aerospike.log
{ycsb}-read
Apr 09 2018 21:33:21
               % > (ms)
slice-to (sec)      1      2      4      8     16     32     64    128    ops/sec
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ----------
21:41:42   501  15.51   8.10   0.08   0.00   0.00   0.00   0.00   0.00     8852.5
21:50:04   502  14.92   7.80   0.07   0.00   0.00   0.00   0.00   0.00     8833.4
21:58:25   501  14.17   7.46   0.07   0.00   0.00   0.00   0.00   0.00     8859.2
22:06:47   502  14.58   7.64   0.08   0.00   0.00   0.00   0.00   0.00     8849.0
22:15:08   501  15.34   8.01   0.08   0.00   0.00   0.00   0.00   0.00     8857.5
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ----------
     avg        14.90   7.80   0.08   0.00   0.00   0.00   0.00   0.00     8850.0
     max        15.51   8.10   0.08   0.00   0.00   0.00   0.00   0.00     8859.2
```
 
Using asloglatency collect the write histogram for the drive from the namespaces that contain the drive measured. 
It is possible the drive has multiple partitions and each partition is used by a different namespace.
 
```
[root@Dell6]# asloglatency -f -3000  -t 500 -n 8 -e 1 -h {ycsb}-write -l /var/log/aerospike.log
{ycsb}-write
Apr 09 2018 21:33:41
               % > (ms)
slice-to (sec)      1      2      4      8     16     32     64    128    ops/sec
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ----------
21:42:02   501  26.52  12.54   0.87   0.00   0.00   0.00   0.00   0.00     2951.0
21:50:24   502  25.70  12.14   0.86   0.00   0.00   0.00   0.00   0.00     2946.5
21:58:45   501  24.78  11.69   0.87   0.00   0.00   0.00   0.00   0.00     2954.1
22:07:07   502  25.41  11.99   0.86   0.00   0.00   0.00   0.00   0.00     2952.0
22:15:28   501  26.43  12.51   0.87   0.00   0.00   0.00   0.00   0.00     2952.3
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ----------
     avg        25.77  12.17   0.87   0.00   0.00   0.00   0.00   0.00     2951.0
     max        26.52  12.54   0.87   0.00   0.00   0.00   0.00   0.00     2954.1
``` 

Using asloglatency collect the write storage histogram for drive from the namespaces that contain the drive being measured.  
In this example  the drive is being used by only one namespace. It is possible the drive has multiple partitions and each partition is used by a different 
namespace. The n option is set to 19 in this example. For this histogram it can go as high as 26. It needs to be set high enough to see the bucket that goes to 0.  
```
asloglatency -f -60  -t 10 -n 19 -e 1 -h {ycsb}-device-write-size -l /var/log/aerospike.log 
 
[root@Dell6]# asloglatency -f -3000  -t 500 -n 19 -e 1 -h {ycsb}-device-write-size -l /var/log/aerospike.log
{ycsb}-device-write-size
Apr 09 2018 21:31:20
               % > (bytes)
slice-to (sec)      1      2      4      8     16     32     64    128    256    512   1024   2048   4096   8192  16384  32768  65536 131072 262144    ops/sec
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ----------
21:39:42   502 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  85.72  85.72  71.43  57.14  42.92  28.45  14.27   0.00   0.00     5907.9
21:48:03   501 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  85.74  85.74  71.48  57.12  42.93  28.45  14.25   0.00   0.00     5923.1
21:56:25   502 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  85.73  85.73  71.47  57.12  42.92  28.41  14.23   0.00   0.00     5919.0
22:04:46   501 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  85.67  85.67  71.37  57.06  42.89  28.47  14.28   0.00   0.00     5925.7
22:13:08   502 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  85.72  85.72  71.44  57.11  42.86  28.43  14.26   0.00   0.00     5907.2
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ----------
     avg       100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  85.72  85.72  71.44  57.11  42.90  28.44  14.26   0.00   0.00     5916.0
     max       100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  85.74  85.74  71.48  57.14  42.93  28.47  14.28   0.00   0.00     5925.7
``` 
 
Using asloglatency collect the read storage histogram for drive from the namespaces that contain the drive being measured.
In this example the drive is being used by only one namespace. It is possible the drive has multiple partitions and each partition 
is used by a different namespace. The n option is set to 19 in this example. For this histogram it can go as high as 26.  
It needs to be set high enough to see the bucket that goes to 0. It is possible the drive has multiple partitions and each 
partition is used by a different namespace.

```
asloglatency -f -60  -t 10 -n 19 -e 1 -h {ycsb}-device-read-size -l /var/log/aerospike.log 
 
[root@Dell6]# asloglatency -f -3000  -t 500 -n 19 -e 1 -h {ycsb}-device-read-size -l /var/log/aerospike.log
{ycsb}-device-read-size
Apr 09 2018 21:31:30
               % > (bytes)
slice-to (sec)      1      2      4      8     16     32     64    128    256    512   1024   2048   4096   8192  16384  32768  65536 131072 262144    ops/sec
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ----------
21:39:52   502 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  83.98  68.29  52.91  38.47  25.53  12.67   0.00   0.00    10407.5
21:48:13   501 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  83.99  68.27  52.92  38.50  25.52  12.65   0.00   0.00    10436.5
21:56:35   502 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  83.95  68.25  52.91  38.53  25.54  12.67   0.00   0.00    10439.2
22:04:56   501 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  83.95  68.24  52.84  38.45  25.49  12.65   0.00   0.00    10448.3
22:13:18   502 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  83.96  68.27  52.88  38.47  25.51  12.63   0.00   0.00    10408.4
-------------- ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ------ ----------
     avg       100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  83.97  68.26  52.89  38.48  25.52  12.65   0.00   0.00    10427.0
     max       100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00 100.00  83.99  68.29  52.92  38.53  25.54  12.67   0.00   0.00    10448.3
 
``` 

 Capture iostat data during the peak load period. In this example the nvme0n1 drive is being used. The numbers that will be 
 used to align the cluster drive utilization between the cluster and ACT are the r/s, w/s, rkB/s, and the wkB/s.  

```
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
nvme1n1           0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdc               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdd               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
nvme0n1      0.00     0.00 11806.00 1547.50 344970.25 198080.00    81.33     5.45    0.42    0.46    0.05   0.03  41.35
```
 
Collect defrag related data from the Aerospike log from a node. This data will be used to determine the large block 
reads for ACT.  Determine whether or not the database is in steady state. Generally the defrag reads and the total write 
blocks should be equal when the cluster is in steady state.  There are cases when a cluster is in steady steady state 
and the total write blocks per second does not equal the defrag reads per second. In this example the total write 
blocks per second is 1543.(see below)  The defrag reads per second is 1172.8. The reason for the difference is the 
post-write-queue, which is a recently written cache. When transactions hit that cache, storage load is not created.
The post write queue acts like a cache while the write blocks are in the queue. If use count for a write block goes to 
zero while it is in the post write queue,  the block does not have to be defraged. The block can be made available immediately. 
Turning the detail for `drv_ssd` allows additional information to be logged regarding write blocks and defrag.  
The value direct frees represents all the blocks that did not have to go through the defrag process. Below the number for direct frees is 369.2. 
The sum of direct frees and defrag reads should be roughly equal to the total write blocks per second. The sum of the defrag reads 
and the direct frees is 1542 and the value for total write blocks is 1543.        

Make sure to save  at least one full  set of data from all the drives and partitions in the namespace for the node.  This example only has a single drive per node without any partitions.  Use the following command to sa
```
tail -n  /var/log/aerospike.log | grep defrag > defrag.out 

Apr 10 2018 07:28:02 GMT: INFO (drv_ssd): (drv_ssd.c:2135) {ycsb} /dev/nvme0n1: used-bytes 1374155776 free-wblocks 12193333 write-q 0 write (65885408,1543.0) defrag-q 36 defrag-read (53996089,1172.8) defrag-write (17909611,406.8)
 
Apr 10 2018 07:28:02 GMT: DETAIL (drv_ssd): (drv_ssd.c:2140) {ycsb} /dev/nvme0n1: defrag-io-skips (3650,0.2) direct-frees (11873218,369)
``` 
 
## Configuring ACT

### Initial changes to the basic ACT configuration file

Adding the data to the ACT configuration file. Below is a sample of the basic configuration file that was generated earlier.  
 
Using the read and write storage histogram averages,  add the following lines to the ACT configuration.  The last value in the histogram lines needs to be 0.

```
# Data from device-read-size histogram
storage-read-histogram:  100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00, 83.97, 68.26,  52.89,38.48,25.52,12.65,0.00   
 
#Data from device-write-size histogram
storage-write-histogram:  100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,85.72,85.72,71.44, 57.11,42.90,28.44,14.26,0.00     
```

Modify the record-bytes value to 0.

```
record-bytes: 0 
```

There will be additional changes to the ACT configuration that will be made during the testing process.

### Determine the Read Ratio and Write Ratio for ACT

Using the read and write storage histograms, take the totals for read/write ops from the storage read and write 
histograms above. The read OPS are 10427 and the write OPS are 5916. The storage histogram operations column for 
reads and writes account for replication factor, read-modify-write operations, and commit to device mode. Those items 
do not have to be accounted for separately. Calculating the read/write ratios will depend 
on how the drives are used in the cluster. The following are a list of difference scenarios and the calculations for 
the read/write ratio.
        
In the rare case that there is one drive per namespace per node, the following are the formulas for the read and write ratios for 
testing using the storage histogram data. This is the scenario that fits this example. 
For this test we simplified this process by just using the read OPS and the write OPS in the storage histogram.  
Depending on the drive configuration this short method is not always possible.

```
    read_ratio = read ops/(storage histogram read OPS + storage histogram write OPS)
    write_ratio = write ops/(storage histogram read OPS + storage histogram write OPS)
```

Aerospike recommends splitting a single raw device into multiple logical devices, in order to gain parallelism.

With multiple drives per namespace, the ratios are calculated the same. 
The final operations used in the ACT configuration file will have to be divided by then number of drives. 

```
read_ratio = read ops/(storage histogram read OPS + storage histogram write OPS)
write_ratio = write ops/(storage histogram read OPS + storage histogram write OPS)
```

If the drive a drive is split between multiple namespaces the ratios appears more complex because the drive 
is shared by many namespaces.  Since the workload that ACT generates is very much like the workload for a drive in a 
single namespace, the solution for spreading drives across multiple namespaces is to use multiple instances of 
ACT for the test.  A separate ACT configuration file will be created for each instance of ACT that will run.  
For example, a 3 node cluster contains one drive on each node and 3 namespaces that share part of each drive. 
The following 

```
read_ratio_ns1  = read ops/(storage hist ns1 read OPS + storage hist ns1 write OPS)
write_ratio_ns1 = write ops/(storage hist ns1 read OPS + storage hist ns1 write OPS)

read_ratio_ns2  = read ops/(storage hist ns2 read OPS + storage hist ns2 write OPS)
write_ratio_ns2 = write ops/(storage hist ns2 read OPS + storage hist ns2 write OPS)
    
read_ratio_ns3  = read ops/(storage hist ns3 read OPS + storage hist ns3 write OPS)
write_ratio_ns3 = write ops/(storage hist ns3 read OPS + storage hist ns3 write OPS)
```

If there are multiple namespaces per drive and multiple drives per namespace, the ratios can again be broken down by namespace.  
The data for each namespace will be used in separate ACT configuration files.  Since there are multiple drives on each 
node in the cluster for this case the resulting read and write operations will have to be divided by the number of drives.

```
read_ratio_ns1  = read ops/(storage hist ns1 read OPS + storage hist ns1 write OPS)
write_ratio_ns1 = write ops/(storage hist ns1 read OPS + storage hist ns1 write OPS)

read_ratio_ns2  = read ops/(storage hist ns2 read OPS + storage hist ns2 write OPS)
write_ratio_ns2 = write ops/(storage hist ns2 read OPS + storage hist ns2 write OPS)
    
read_ratio_ns3  = read ops/(storage hist ns3 read OPS + storage hist ns3 write OPS)
write_ratio_ns3 = write ops/(storage hist ns3 read OPS + storage hist ns3 write OPS)
```

### Determine the number OPS per second

The source of the read and write operations per second will come from the storage histogram.  These operations 
represent the operations that are actually generating load on the drives and are more meaningful for creating the workload for ACT.  

The storage histograms provide the data sizes and operations per second by namespace for a node in a cluster. Looking 
at the far right column of the histogram you will see the a the column titled ops/sec. From the data taken earlier the 
average reads sampled is 10470 reads per second  and the average writes sampled is 5916 per second.  In this example 
there is one drive per node per namespace.  So the number of read and write operations would be exactly the numbers you 
see in the histogram.  The following are the equations for calculating operations under different scenarios. 

First calculate the total storage operations per node per namespace.

```
    Total storage ops per node per namespace = 
                Storage read ops per node per namespace +
                Storage write ops per node per namespace
```

If there is one drive per node per namespace, calculate the read and write ops.

```
read_ops = read_ratio * total storage ops per node per namespace.
write_ops = write_ratio * total storage ops per node per namespace.
```

If there were multiple drives per node per namespace, the storage read operations and storage  write operations would 
have to be divided by the number of drives on that node for the namespace. 

```
read_ops = (read_ratio * total storage ops per node per namespace)/number of drives
write_ops = (write_ratio * total storage ops per node per namespace/number of drives
```

If a single drive in a namespace per node  is shared between multiple namespaces, then each namespace 
is handled independently and multiple ACT configuration files will be used. The formula is basic the same as a single namespace.

```
    For each namespace that uses the drive.
        read_ops = read_ratio * total storage ops per node per namespace.
write_ops = write_ratio * total storage ops per node per namespace.
```

If there are multiple drives per namespace and multiple namespaces per drive.
    
For each namespace that uses the drive,

```
read_ops = (read_ratio * total storage ops per node per namespace)/number of drives
write_ops = (write_ratio * total storage ops per node per namespace/number of drives
```

The baseline configuration will utilize these initial read and write operations per second.  
As testing progresses on the new drives, the faster drives may be able to handle more operations per second.  

Increase the total number operations by 2x or 3x and recalculate the operations per second for test.

## Running ACT

Salt the drive with `actprep`
```
        nohup ./actprep /dev/nvme0n1
```

Run ACT  

If you are running with a single namespace per drive, you will run only one instance of ACT. 
If the configuration was based on multiple namespaces per drive you will have to run an instance of ACT for each 
namespace.  Use the following command to run ACT.

```
nohup act  act_basline.txt  >  act_output.log &
```

Capture IOSTAT
```
nohup iostat -p -x 2 > iostat.out &
```

## Analyzing Results and Making Adjustments

The first step for aligning ACT and the cluster is to get the large block reads and writes to align by using iostat and iosnoop.   The important numbers to look at on on iostat are w/s(writes per second) and r/s(reads per second). In general the w/s value represents the large block writes allocated to a drive assuming that commit-to-device mode is not enabled.   In this example there is only one drive per node per namespace.  In this case the iostat w/s value should be equal to the large block reads and writes.  Looking at the iostat from the cluster the w/s value is 1548. 

```
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.74    0.00    1.23    6.79    0.00   91.25
 
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
nvme1n1           0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
nvme0n1           0.00     0.00 11788.50 1548.00 340545.00 198144.00    80.78     4.95    0.38    0.43    0.05   0.03  41.65
```

Next look at the iostat capture from the running ACT test.

```
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.62    0.00    0.49    5.30    0.00   93.58
 
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
nvme1n1           0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
nvme0n1           0.00     0.00 12172.50 1702.00 388503.50 217856.00    87.41     2.71    0.24    0.26    0.06   0.05  75.75
``` 
 
The w/s value from the ACT test is 1702 which is higher than the w/s value from the cluster.  A couple of ACT 
configuration items that can be used to adjust large block reads and large block writes are `lbr-req-offset` and `lbw-req-offset`.  
These values can be used to adjust the large block reads up or down.   A value off 100 represents 100 percent. Values great than 
100 increase the reads or writes values less than 100 decrease the reads and writes.  Setting the lbw-req-offset to 90 and 
rerunning ACT generated the following iostat.
 
```
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
nvme1n1           0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
nvme0n1           0.00     0.00 12172.00 1531.50 385051.50 196032.00    84.81     2.55    0.23    0.25    0.06   0.05  72.00
```

Modifying the `lbw-req-offset` brings the large block writes into alignment with the cluster. 
Since ACT is generating 1531 large block writes. It will be generating an equivalent number large block reads. 
Recall from the defrag data taken earlier the number of large block reads being done by the node in the cluster is 1172.

```
Apr 10 2018 07:28:02 GMT: INFO (drv_ssd): (drv_ssd.c:2135) {ycsb} /dev/nvme0n1: used-bytes 1374155776 free-wblocks 12193333 write-q 0 write (65885408,1543.0) defrag-q 36 defrag-read (53996089,1172.8) defrag-write (17909611,406.8)
```

The ACT value has to be reduced.  The cluster is generating fewer large block reads because of the direct frees 
being done. Using the information collected earlier, the direct-frees and total large block writes should be 
added to the ACT configuration file. The following two lines should be added and the ACT test be should be rerun.
 
```
total-lbw-per-sec: 1543
direct-frees: 369
```

After making the changes and rerunning the ACT test, iostat generated the following results. 
 
```
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.55    0.00    0.49    5.73    0.00   93.23
 
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
nvme1n1           0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
nvme0n1           0.00     0.00 12803.50 1532.00 467442.50 196096.00    92.57     2.92    0.25    0.27    0.07   0.06  79.25
```

The r/s value is 12803.  At this point it is hard to confirm how many large block reads are being done with ACT and 
how many regular reads.  The tool iosnoop from perf-tools can help identified the number of large block reads being done. 
Run `iosnoop` on the cluster node and the act server for ten second each using the following command.

```
timeout 10 ./iosnoop | grep 131072 > ios.out
```

Run `iosnoop` 

The following commands can be used to determine the number of large block reads and writes done on the cluster node and the ACT server. 
If you don’t want to run `iosnoop` on the cluster you can look at the defrag read value in the Aerospike log and multiply 
that number by 10 for the total large blocks read in 10 seconds.   
 
```
cat ios.out | grep R > iosread.out
cat ios.out | grep WS > ioswrite.out
```

The number of lines in each of these files represents the number of large block reads and writes done in 10 seconds.  
Compare the numbers from the ACT server and the cluster. The cluster is generating 11194 large block reads in 10 seconds 
and ACT is generating 12062. The large block reads have already been adjusted for the direct frees. 
The minor adjustments can be managed by the `lbr-req-offset`.   Adding the following line to reduce the reads by about
10% should being the cluster and ACT into alignment.

```
        lbw-req-offset: 093
```

The r/s value are now 10800 and closely aligned with the cluster. 

Now that you have alignment between the IOSTATs,  you can think of this baseline  configuration as 1x. The baseline configuration can be used to begin testing new drives.

A question that might come up that the example doesn’t describe how to manage when a drive is partitioned and shared across multiple namespaces. 
The types of loads that ACT creates are similar to what you would see in a namespace. 
If a drive is shared, then a test with multiple namespaces should be run. Using the process above, collect the data for multiple namespaces. 
Generate a baseline configuration file for each namespace.  Run a separate instance of ACT for each namespace  simultaneously to generate the load on the drive.    

## Testing New Drives

The rest of the process will be narrowing the drive selection down from a range of possible new drives.  
Start the narrowing process by first running  basic act tests on all drives of interest. If the basic test results 
exist on the Aerospike website, then isn’t necessary to run the basic tests. The directions for running ACT can be 
found on github at `https://github.com/aerospike/act`.  

After running the basic ACT tests, eliminate the drives that are non performant. Begin running the ACT tests with 
the baseline configuration file on the remaining drives. Capture the ACT log and iostat for all of the drives tested. 
Compare the ACT latency information to the SLAs for the use case. Compare the iostat data to the original cluster
data to confirm the drive is being utilized with the load of a drive on the production cluster. Eliminate any drives that 
don’t meet the minimum SLA requirements. For drives that meet the SLA and are under utilized, incrementally adjust the 
reads and writes and rerun the ACT until the drives no longer meet the SLA.  The reads and writes should be adjusted 
according to the read write ratios calculated in the earlier steps of this process.   

Now that the drive performance has been determined for candidate drives, the final step will be to test the 
selected drive in a development cluster to see how they perform with  Aerospike. This is an important step because 
the performance of the new drive may change the quantity of drives that are used in a cluster. The quantity will 
change how the drives are configured for post write queue and defrag. For example the new drives may need to be 
partitioned when the older drives had no partitions.  Run the tests in a development Aerospike cluster and compare 
the latency and performance to the production cluster.  

## Final Thoughts

The new changes to the ACT tool have extended its ability to closely mirror the activity on a Flash device in a production Aerospike cluster.
ACTs new flexibility provides a better tool to benchmark and choose the right drive for your SLA. 
This paper details the technical steps necessary to evaluate the performance of new drives for an existing Aerospike cluster. 
The process may uncover multiple drives that meet your requirements. Other technical factors such as form factor, 
size, and drive writes per day will also help the selection process. In addition to the technical factors, 
price, availability, and product lifecycle may also play a role in the SSD selection process.   


 
## Appendices

### Load Generator Commands

Commands for loading data into Aerospike. These sizes and key counts were specifically used to utilize the benefits of the post write queue. 

```
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s1 -k 320000 -b 1 -o B:512   -w I -latency 11,1  -z 30 > load1.out 2> load1.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s2 -k 160000 -b 1 -o B:1024  -w I -latency 11,1  -z 30 > load2.out 2> load2.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s3 -k 80000 -b 1 -o B:2048  -w I -latency 11,1  -z 30 > load3.out 2> load3.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s4 -k 40000 -b 1 -o B:4096  -w I -latency 11,1  -z 30 > load4.out 2> load4.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s5 -k 20000 -b 1 -o B:8192  -w I -latency 11,1  -z 30 > load5.out 2> load5.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s6 -k 10000 -b 1 -o B:16384 -w I -latency 11,1  -z 30 > load6.out 2> load6.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s7 -k 10000 -b 1 -o B:32768 -w I -latency 11,1  -z 30 > load7.out 2> load7.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s8 -k 10000 -b 1 -o B:65536 -w I -latency 11,1  -z 30 > load8.out 2> load8.err &
```

Commands for running the workload against Aerospike.

```
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s1 -k 320000 -b 1 -o B:512   -w RU,75 -latency 11,1 -g 5000 -z 30 > tst1A1.out 2> tst1A1.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s3 -k 80000 -b 1 -o B:2048  -w RU,75 -latency 11,1 -g 5000 -z 30 > tst3A1.out 2> tst3A1.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s4 -k 40000 -b 1 -o B:4096  -w RU,75 -latency 11,1 -g 5000 -z 30 > tst4A1.out 2> tst4A1.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s5 -k 20000 -b 1 -o B:8192  -w RU,75 -latency 11,1 -g 5000 -z 30 > tst5A1.out 2> tst5A1.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s6 -k 10000 -b 1 -o B:16384 -w RU,75 -latency 11,1 -g 5000 -z 30 > tst6A1.out 2> tst6A1.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s7 -k 10000 -b 1 -o B:32768 -w RU,75 -latency 11,1 -g 5000 -z 30 > tst7A1.out 2> tst7A1.err &
nohup ./run_benchmarks -n ycsb -h 192.168.201.202 -p 3000 -s s8 -k 10000 -b 1 -o B:65536 -w RU,75 -latency 11,1 -g 5000 -z 30 > tst8A1.out 2> tst8A1.err &
```

### New ACT 4.0 configuration parameters

**update-pct**
Simulate the device load you would see if this percentage of write requests were
updates, as opposed to replaces. Updates cause the current version of a record
to be read before the modified version is written, while replaces do not need to
read the current version. Therefore a non-zero update-pct will generate a
bigger internal record-sized read rate. E.g. if read-reqs-per-sec is 2000 and
write-reqs-per-sec is 1000, the internal read-req rate will be somewhere between
2000 (update-pct 0), and 2000 + 1000 = 3000 (update-pct 100).
The default update-pct is 0.
 
**defrag-lwm-pct**
Simulate the device load you would see if this was the defrag threshold. The
lower the threshold, the emptier large blocks are when we defragment them (pack
the remaining records into new blocks), and the lower the "write amplification"
caused by defragmentation. E.g. if defrag-lwm-pct is 50, the write
amplification will be 2x, meaning defragmentation doubles the internal effective
write rate, which for ACT is manifest as the large-block read and write rates.
The default defrag-lwm-pct is 50.
 
**commit-to-device**
Flag to model the mode where Aerospike commits each record to device
synchronously, instead of flushing large blocks full of records.  This causes a
device IO load with many small, variable-sized writes.  Large block writes (and
reads) still occur to model defragmentation, but the rate of these is reduced.
The default commit-to-device is no.
 
**commit-min-bytes**
Minimum size of a write in commit-to-device mode. Must be a power of 2. Each
write rounds the record size up to a multiple of commit-min-bytes. If
commit-min-bytes is configured smaller than the minimum IO size allowed on the
device, the record size will be rounded up to a multiple of the minimum IO size.
The default commit-min-bytes is 0, meaning writes will round up to a multiple of
the minimum IO size.
 
**tomb-raider**
Flag to model the Aerospike tomb raider. This simply spawns a thread per device
in which the device is read from beginning to end, one large block at a time.
The thread sleeps for tomb-raider-sleep-usec microseconds between each block.
When the end of the device is reached, we repeat, reading from the beginning.
(In other words, we don't model Aerospike's tomb-raider-period.)
The default tomb-raider is no.
 
**tomb-raider-sleep-usec**
How long to sleep in each device's tomb raider thread between large block reads.
The default tomb-raider-sleep-usec is 1000, or 1 millisecond.

**storage-read-histogram**  
This is the histogram data from the read storage histogram in the Aerospike log. 
Each column in the histogram  represents an increase in object size by a power of 2. 
The values are between 0.0 and 100.0. The value represents the percent greater than.  If the 3rd bucket is 90.0. 
The percent of objects greater than 8 bytes is 90%.  The values are all comma separated and the final value should be 0. 
ACT uses this histogram to generate a range of objects that are used by ACT.

**storage-write-histogram**
This is the histogram data from the write storage histogram in the Aerospike log. 
Each column in the histogram  represents an increase in object size by a power of 2. 
The values are between 0.0 and 100.0. The value represents the percent greater than. If the 3rd bucket is 90.0. 
The percent of objects greater than 8 bytes is 90%.  The values are all comma separated and the final value should be 0.  
ACT uses this histogram to generate a range of objects that are used by ACT.

**total-lbw-per-sec**
Database writes are placed in large blocks. When the large block fills, it gets flushed to disk.  When the large 
block use count goes to zero it gets goes through the defrag process and is made available again.  The total large blocks 
writes value includes both large blocks writes to disk and the defrag writes to disk.  
This value can be found on the drv_ssd line in the Aerospike log. 

**direct-frees**
When Aerospike utilizes the post write queue efficiently, some of the large blocks never have to go 
through the defrag process.  The blocks are simply added back to the available queue.  This value can be found in the 
Aerospike log after turning the detail on for the drv_ssd context.  Be careful turning on detail.  
Make sure it is detail for the drv_ssd only. Turning detail on for all contexts can fill the log fast.  

**lbr-req-offset**
This is an adjustment to large block reads.  This allow the large block reads to be adjusted up or down 
independent of large block writes. The default number for this value is 100 and represents a percent of the large 
block reads. Values below 100 reduce the large block reads. Values above 100 increase the large block reads. The 
large block reads can be fined tuned to match the large block reads of a cluster. 
This currently only works with the storage histograms. 

**lbw-req-offset**
This is an adjustment to large block writes.  This allow the large block writes to be adjusted up or down 
independent of large block reads. The default number for this value is 100 and represents a percent of the large 
block writes.  Values below 100 reduce the large block writes. Values above 100 increase the large block writes. 
The large block writes can be fined tuned to match the large block writes of a cluster. This currently only works with the storage histograms. 

**write-size-offset**
This value represents an adjustment to the write record sizes that are created by the storage histogram. Since the sizes are based on powers of two the actual object size may be in between two of the values in the storage histogram. Using this value the object sizes can be adjusted up or down to bring the ACT results in line with the performance of a drive in an Aerospike cluster. The default number for this value is 100 and represents a percentage size a write record. Values below 100 reduce the write record sizes. Values above 100 increase the write record sizes.
 
**read-size-offset**
This value represents an adjustment to the read record sizes that are created by the storage histogram. Since the sizes are based on powers of two the actual object size may be in between two of the values in the storage histogram. Using this value the object sizes can be adjusted up or down to bring the ACT results in line with the performance of a drive in an Aerospike cluster. The default number for this value is 100 and represents a percentage size a read record. Values below 100 reduce the read record sizes. Values above 100 increase the read record sizes.

### Final ACT Configuration used in this example

```
##########
act config file for testing 1 device(s) at non-standard load
##########
 
# comma-separated list:
device-names: /dev/nvme0n1
 
# mandatory non-zero:
num-queues: 4
threads-per-queue: 35
test-duration-sec: 7200
report-interval-sec: 1
large-block-op-kbytes: 128
 
# Old style ACT
record-bytes: 0
 
# Data from live cluster.  Last number must be 0
storage-read-histogram: 100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,84.00,68.36,52.95,38.48,25.56,12.68,0.00
storage-write-histogram: 100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,100.00,85.78,85.78,71.59,57.17,42.91,28.46,14.22,0.00
 
# Effects of post-write-queue. Data comes from drv_ssd in the Aerospike log.
direct-frees: 300
total-lbw-per-sec: 1527
 
# Settings to modify the read size and write size for the data from  the storage histogram. The number is between 1 and 100.
#write-size-offset: 090
#read-size-offset: 090
 
# Independently adjust the large block reads and large block writes to align with server data.
lbr-req-offset: 090
lbw-req-offset: 093
 
# Operations per second # usually non-zero:
read-reqs-per-sec: 10470
write-reqs-per-sec: 5957
 
# yes|no - default is no:
microsecond-histograms: no
 
# noop|cfq - default is noop:
scheduler-mode: noop
```


### Hardware used in this specific example

In order to create the data for this example, the following equipment and tools were used. The tests utilized a 3 node Aerospike cluster. A fourth  server was used for running ACT tests. A separate server was also used to generate the load on the Aerospike database. 

Aerospike Cluster:
- Servers:  3 Dell PowerEdge R730xd
- Aerospike Version: 4.0.0.1-26
- OS: Centos 7.2
- SSD: 3 Samsung 1725a

ACT Server:
- Server: 1 Dell PowerEdge R730xd
ACT Version: Modified 4.x soon to release.
OS:  Centos 7.2
SSD: 1 Samsung 1725a.

Load Generator:
- Server: 1 Dell PowerEdge R730xd
- Client:  Aerospike Java Client.
- Load Generator:  Aerospike Java Benchmark.
    
See the Appendix for load generator commands. These commands were developed to create a workload that takes advantage 
of the post write queue caching and direct frees of write blocks.
