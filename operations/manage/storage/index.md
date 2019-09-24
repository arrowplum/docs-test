---
title: Managing Storage Capacity
description: Learn how to manage storage properly when high volumes of data are generated quickly.
---

Storage is always an issue when high volumes of data are generated quickly.  Under normal circumstances, you will have policies in place that allow the database to make automatic decisions about how to handle volumes of data that exceed capacity.

### Eviction

**Eviction** is the act of removing data from the cluster when additional storage space is required. When the high watermark for either memory or storage is exceeded, nodes will begin to evict data based on what is the soonest to expire. Note that eviction is different than expiration but is still based on the time-to-live (TTL). If no TTL is set, evictions do not have a basis on which to determine which data to delete and you may run out of space when high volumes of data are generated quickly. For such cases, it is important to set the TTL, even if it is set at a very long time. For systems expecting burst of high volumes of new data which may exceed memory or disk capacity, it is recommended to keep TTLs homogeneous within a namespace to make sure data that is most likely to expire first is evicted first.

- You can check if you are evicting records by using one of the methods below. The eviction counter is reset every time the server is restarted. 

- Use the info command within [asadm](/docs/tools/asadm). This will report back a list of how much free disk and memory exist for each namespace. In addition it will print the currently configured limits to the water marks for both memory and disk.

    If for some reason, asadm is unable to reach the node(s), look for messages in the log that show you may be evicting data by using the following command. You should log into each node and run the following command.

    ```
    $ sudo grep -e "hwm_breached" -e "stop_writes" /var/log/aerospike/aerospike.log
    ```

    If any of these exist in the log, then you may be in a state where the system is attempting to evict data.

### Expiring Data that has Reached TTL

**Expiration** is the act of removing data from the cluster because it has aged beyond its TTL (time-to-live). If a record is updated, the TTL will be extended by the current TTL setting (or by the programmable value). If a record is read only (no write), the TTL will not be extended. For server version 3.10.1 and above, you can retain the TTL of the record when updating it by setting the TTL to a value of -2. See respective client API docs for details.

{{#warn}}Check that the system clocks on all your nodes are synchronized. If the clocks are out of sync, records may not expire correctly.{{/warn}}

### Setting Time-to-live (TTL) for Data Expiration

TTL is configured by default for the entire namespace but can be overridden for specific records by the application.  

You must be very careful in the setting of the default TTL in the configuration file. Carefully consider the following behaviors

default-ttl value |  application TTL value |  result 
--- | --- | ---
\>=0  | -2 | record TTL will not be updated when record is updated but will use the default-ttl value at record creation.  Available for server version 3.10.1 and above.
0  | 0  | data will never expire or be deleted by the system. Must be manually removed.
0  | >0 | data will expire with the application specified TTL value
\>0 | 0  | data will expire with the DEFAULT TTL value
\>0 | >0  | data will expire with the application specified TTL value
\>=0  | -1 | data will never expire, regardless of DEFAULT TTL value

For systems where high data volumes come in burst or are not fully predictable and can cause memory or storage levels to grow beyond capacity, we do not recommend setting the default-ttl to zero. If you do set default-ttl to zero, make sure that the application developers are aware that they must set a TTL for any data that should expire.

### Aerospike's Automatic Storage Reclamation Processes

Aerospike Database uses these methods to reclaim/manage storage:

* **Defragmentation**  - On an ongoing basis, the database reclaims storage that is no longer needed from the SSD.

* **Expiration** - The configuration files specify when older data can be deleted – typically 90 days.  

Under normal circumstances, defragmentation and expiration are ongoing, to ensure that storage is always available.  When storage levels fall too low despite the defragmentation and expiration process, the database proceeds to:

* **Eviction** - Deleting data that is nearing its TTL expiration. In other words, accelerating data expiration.

Here's how it works:

- Defragmentation continously happens in the background. Look for defrag related [configuration variable](/docs/reference/configuration) for details.
- Below all of [high-water-disk-pct](/docs/reference/configuration/index.html#high-water-disk-pct)/[high-water-memory-pct](/docs/reference/configuration/index.html#high-water-memory-pct)/[mounts-high-water-pct](/docs/reference/configuration/index.html#mounts-high-water-pct) – expiration, NO eviction
- Above any of high-water-disk-pct/high-water-memory-pct/mounts-high-water-pct – expiration, eviction
- Above [stop-writes-pct](/docs/reference/configuration/index.html#stop-writes-pct) or below [min-avail-pct](/docs/reference/configuration/index.html#min-avail-pct)– read requests will still be performed, but no more data is written to the database (incoming write requests fail but replica writes and incoming data from migration would still go through)

To defragment a SSD while running a big data database with high traffic, you need half of the disk free.  Data is being read/written from all parts of the disk at all times, and as a result, we need more defragmentation space than you might be accustomed to.

The high water mark for memory and disk specifies the level at which we start evicting data, by default 50% for disk and 60% for memory.  The stop-writes-pct percentage (default 90%) for memory/min-avail-pct percentage (default 5%) for disk specifies the point at which the node stops accepting new write requests.  If you have the default configuration values and have sized the cluster correctly, your SSDs should never get over-full and this problem should never arise.

However if you have changed the default settings and if your data grows beyond the cluster's capacity, you may exceed your storage capacity if:

1. The system could not evict data fast enough and eventually the data exceeds the limit.
2. At 90% memory usage, the system hits the (default) stop write limit and the database stops writing new records but continues to process prole writes as well as potential incoming migration writes, so the data may continue to grow.
3. Note that for server version prior to 3.15, the stop-write flag does depend on the ongoing Namespace Supervisor cycle to complete. Thus, depending on the duration of the cycle, it could take longer to set the flag to true and the replica write and migrated data could trigger the server towards out-of-memory (or completely out of available storage).


### Nodes won't Start if there is not Enough Storage

If the database does not have enough contiguous storage to start and does not have enough space to defragment to get the space it needs, it will not start. 

For persistence files for in-memory databases, you specify the size of the persistence file (whereas when using an SSD, you just use the entire SSD).  The persistence file size can also run out of space and the same rules apply as for SSDs.

### What Happens when a Namespace runs low on Storage?

When a namespace can no longer write data, you will see error messages in the log.  For example, this example message:

```
Sep 05 2012 21:28:48 GMT: INFO (namespace): (base/namespace.c:458) {test} lwm breached true, hwm_breached true, stop_writes true, memory sz:22971755648 nobjects:358933683 nbytesmem:0 hwm:23192823808 sw:34789232640, disk sz:216122189312 hwm:216116854784 sw:341237137408
```

means that the namespace test on that node has reached the high-water-mark for either disk or memory and the stop-writes-pct percentage.  As a result, the namespace can no longer accept write requests. Additional messages that look like this:

```
Sep 05 2012 21:28:48 GMT: INFO (rw): (base/thr_rw.c:2300) writing pickled failed 8 for digest 7318ad7422e51009
```
happen as the result of the stop_writes being true either on this node or others. You can resolve this issue by adjusting configuration parameters:

1. Speed up your current eviction rate by reducing the memory/disk high-water-mark ( high-water-disk-pct/ high-water-memory-pct )
2. Slow down your migration speed, if migrations are happening
3. Increase your defragmentation priority or rate 
4. Increase the stop-writes-pct (% of disk usage above which the database will stop writing new records)

{{#warn}} Increasing the stop-writes-pct parameter should not be done on a permanent basis – you need to find a more permanent solution by reviewing your capacity and ensuring that there is sufficient storage.{{/warn}}

All of these parameters can be changed dynamically in the main Aerospike configuration file on the node.

### Avoiding 0% Available Space

When the previous situation (storage running low) goes on for too long, you will see that the log will show many entries similar to the following:

```
Apr 27 2012 02:53:12 GMT: WARNING (drv_ssd): (storage/drv_ssd.c:1844) could not allocate storage on device /dev/sdb
```

In addition, when the available_pct goes to zero, all the subsequent writes will be reported as success to the client, and the write block buffer will fail when it tries to write to SSD. This should not happen if default min-avail-pct has not been modified.

{{#warn}} Taking a server down increases traffic/data on the other nodes. Do not take any servers down if you are in a data overflow situation.

If only a single node is having problems, because of a hardware problem, then taking down the problematic node may resolve the situation.

The solutions discussed here are short-term, temporary updates.  In the longer term, you need to add capacity to resolve storage overflow problems.{{/warn}} 

### Adjusting Defragmentation

When defragmentation cannot keep up with storage requirements, you may have to increase the defragmentation rate.

### Increase the defrag rate

If defragmentation is not keeping up, you may need to temporarily decrease the [defrag-sleep](/docs/reference/configuration/index.html#defrag-sleep) and increase the [defrag-lwm-pct](/docs/reference/configuration/index.html#defrag-lwm-pct).

If your system has a very high write level, you should set defrag-sleep to zero permanently. 

To do this at next server restart, simply alter the configuration file by adding the following lines to the namespace section, for example:

```
defrag-sleep 500
defrag-lwm-pct 60
```

In order to do this temporarily (until next Aerospike restart), use the following commands:

```
$ asinfo -v "set-config:context=namespace;id=<namespace_id>;defrag-sleep=500"  -h [hostname]
$ asinfo -v "set-config:context=namespace;id=<namespace_id>;defrag-lwm-pct=60"  -h [hostname]
```


### Improved storage utilization

**In Aerospike version 2.7.0 and above, and Aerospike version 3.1.0 and above, an important change has been made to improve storage utilization.**

In previous versions, a record on device consumes a multiple of 512 bytes.  This is especially inefficient for the smallest records (e.g. integer value records).  Recent versions reduce the storage size unit from 512 to 128 bytes. For storage size calculation details, please see the [Capacity Planning Guide](/docs/operations/plan/capacity).

#### Benefits

For new deployments, this storage improvement may affect the result of capacity planning, and allow for smaller and/or fewer storage devices.

In general, more compact storage also translates directly to a reduction in (write-block) storage device IO activity.  This reduction in device write load may noticeably improve read latencies in SSD-based clusters.

If existing deployments upgrade to this version and recover a significant amount of available storage device space, the extra space may be traded away to further improve performance – by reducing defrag-lwm-pct to allow blocks to become more depleted before defragmenting them, device write load can be further decreased.  (Before making such a configuration change, care should be taken to make sure the cluster has reached its new storage equilibrium – see below.)

#### Operational Procedure for Existing Deployments

It is possible to [fast restart](/docs/operations/manage/aerospike/fast_start) on upgrading to this version.  Following either fast or cold restart, the storage statistics (disk-use and available-percent) will initially be unchanged.  As existing records are modified in any way, or moved due to defragmentation, they will pack to the new format.  (Of course new records will be written in the new format.)  The new storage statistics picture will be fully revealed when all existing records have been modified, defragmented, or deleted (including deletion by expiration or eviction). The storage format is not backward compatible. Once a node is upgraded, any downgrading will require a clean-disk and propagation of replica data to the node.


