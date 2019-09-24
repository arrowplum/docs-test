---
title: LSI StorCLI (AKA MegaCLI)

---

While storage systems have moved beyond RAID cards and devices to encompass
high performance direct attach devices such as NVMe devices, RAID devices are still in use.

One of the most popular vendors of RAID cards was LSI, and this configuration guide will help an
LSI based motherboard be configured for optimal speed with Aerospike.


LSI has created a tool to help manage RAID devices called StorCLI (previously MegaCLI). 
You can find the <a href="http://www.avagotech.com/cs/Satellite?pagename=AVG2%2FsearchLayout&locale=avg_en&Search=storcli&submit=search">documentation for StorCLI on their web site.</a> 
This software will let you control many RAID controllers that are based on LSI chipsets.


### LSI&trade; MegaRAID&reg; FastPath&trade;
While StorCLI can be used for many RAID devices, it can be very useful in turning on LSI&trade; MegaRAID&reg; FastPath&trade;. 
Not all RAID devices support FastPath&trade;, but for those that do, the difference in performance can be huge. Several high end RAID controllers from Dell and IBM support this mode, but do not always include the software with their servers. 

In order to turn on FastPath&trade;, you must set the up each drive with the following:

| Parameter | Value |
| --- | --- |
| RAID Level | 0 |
| Write Policy | Write Through* |
| Read Policy | No Read Ahead |
| IO Policy | Direct IO |
| Other | No write to cache if bad BBU |

\* Many DBAs familiar with traditional relational databases are accustomed to using RAID devices that are striped across drives and using a write back strategy. However for best performance you should turn on the LSI FastPath for those devices that support it. Please refer to the [LSI StorCLI Reference Manual](http://docs.avagotech.com/docs/12352476).

### Does FastPath&trade; really work for Aerospike?
A natural question is whether or not it is worth it to do this. Aerospike has conducted testing and found the following tests results using the <a href="http://github.com/aerospike/act">Aerospike Certification Tool (ACT)</a>.

These tests were performed using the exact same hardware:
- Dell R720xd
- PERC H710p RAID controller
- 8 x 200 GB Intel s3700 SSDs 

**Latency measures during torture tests (96,000 reads/sec and 48,000 concurrent writes/second).**

| RAID setting | % >1 ms | % >2 ms | % >4 ms | % >8 ms | % >16 ms |
| --- | --- | --- | --- | --- | --- |
| Default | 22.20 | 13.96 | 4.29 | 0.22 | 0.00 |
| FastPath&trade; | 4.31 | 0.85 | 0.17 | 0.00 | 0.00 |

The latencies for the FastPath&trade; are much better. In general better latency results mean much higher threshold for throughput. 

### Common tasks

After installing the StorCLI package. Here are some common tasks:

#### Get the current settings of all RAID arrays. 
The following command will give you the RAID configuration of all arrays. In this case "/c0" refers to the first RAID controller on the server. This will also give you the model of the SSDs.
```bash
sudo storcli64 /c0 show
```

#### To configure each drive as its own RAID 0 array:
In this example we will create the 17th virtual disk array on the server (16 have already been configured). This will use the 17th drive on the first RAID controller (/c0).

The following command has been tested on the Dell H710p RAID controller, but settings may vary.
```bash
sudo storcli64 /c0 add vd type=raid0 drives=32:17 wt nora direct NoCachedBadBBU 
```
32 is the enclosure number (likely to be the same for yours)

17 is the drive slot, will vary depending on the SSD you wish to set up as RAID 0, but will correspond to its position in the list given by the "show" command above.

#### Confirm the cache configuration
Run the show command to look at all the configured disks:
```bash
sudo storcli64 /c0/dall show all
```
Look for the term "VD LIST" within the output:
```bash
...
VD LIST :
=======

----------------------------------------------------------------
DG/VD TYPE  State Access Consist Cache  Cac sCC       Size Name
----------------------------------------------------------------
0/0   RAID0 Optl  RW     Yes     NRWTD  R   OFF  465.25 GB
1/6   RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
2/7   RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
3/8   RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
4/9   RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
5/10  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
6/11  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
7/12  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
8/13  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
9/14  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
10/15 RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
11/16 RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
12/2  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
13/3  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
14/4  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
15/5  RAID0 Optl  RW     Yes     NRWTD  -   OFF 146.625 GB
16/20 RAID0 Optl  RW     Yes     RaAWBD -   OFF   372.0 GB r0
17/1  RAID0 Optl  RW     Yes     NRWTD  -   OFF 446.625 GB
-------------------------------------------------------------------
```
Note that the just created 17th virtual disk (last on the list) has "NRWTD", which is "No Read Ahead", "Write Through", and "Direct".
