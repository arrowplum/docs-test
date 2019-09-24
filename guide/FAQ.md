---
title: Frequently Asked Questions (FAQ)
description: Frequently Asked Questions (FAQ)
---

- [Definitions](#definitions)
- [General Information](#general)
- [Hardware](#hardware)
- [Database](#database)
- [Storage](#storage)
- [Data Migration and Synchronization](#data)

<a name="definitions">
### Definitions
</a>

**What is Citrusleaf?**

Citrusleaf is Aerospike's former name.  We became Aerospike in the summer of 2012 but you'll see many labels that still refer to citrusleaf or cl.  Over time we'll remove these references.

**What is the definition of [Bin, Cluster, Namespace, Record, Set, Digest]**

See the [Glossary](/docs/dev_reference/glossary.html).

**What is a Distributed Hash Table or DHT**

The Distributed Hash Table is the accumulated information on where data is stored within the cluster. It includes the indexes that are distributed throughout the cluster and how they map to partitions (see “partition” and “partition map” below) and nodes.

**What is the Generation Number?**

When a record is written to cluster, part of the metadata associated with the record is the generation. It is incremented each time the record is altered. The generation is used to resolve any conflicts that may occur if two records have different values.

**What is Migration?**

During normal operation an Aerospike cluster will have some number of copies of the data. The number of copies is defined as the replication factor. If this factor is greater than 1 (2 or more), then multiple copies will be evenly distributed across the cluster. If a node goes down, then some amount of the data will no longer be at the full replication factor. The Aerospike cluster will respond by automatically transferring the data between the nodes to re-establish a full replication factor for all data. The motion of this data is referred to as “migration.”

There is a setting for how many threads to use for migration. Aerospike recommends setting this to “1″ for each node. This will ensure that migrations do not interfere with serving requests.

**What is a Partition?**

“Partitions” are buckets of records that have been grouped together for the purpose of distribution.

In order to evenly divide data between different nodes, all data is mapped to one of 4096 partitions, based on the hash (digest) value. In turn each of these partitions is mapped to the different nodes. If the number of nodes change, the partitions will get remapped and transferred to the appropriate location in a process called “migration.”

Partition Map Every record in an Aerospike cluster will be mapped to one of 4096 partitions. The basis for this mapping is the hash of the key. 4 bytes of the hash are used to determine to which partition any given key belongs. These partitions are then divided among the nodes in the cluster in a partition map.

**What is a Replication Factor?**

Every Aerospike namespace is configured with a replication factor that determines the number of copies of the data. The number of copies is referred to as the “replication factor.” A replication factor of “1″ means that the data is not replicated and does not have a hot backup. Most Aerospike customers use a replication factor of “2.”

**Set** 

An Aerospike “set” is similar to a table in a relational database. One of the big differences is that with Aerospike, you do not need to predefine a schema. You may add bins (or columns) to one record in a set without needing to add them to any other record in the set.

**What is CITRUSLEAF_EPOCH Time?** 

Aerospike compacts dates by subtracting out the CITRUSLEAF_EPOCH time. The epoch is taken as the second before 12:00:01 am, January 1, 2010 GMT. Time in the Citrusleaf epoch can be calculated by the following formula:

```ini
Current time - 1262304000 = Time in CITRUSLEAF_EPOCH
```
 
```ini
#define CITRUSLEAF_EPOCH 1262304000
struct timespec ts;
clock_gettime(CLOCK_REALTIME, &ts);
return ( ts.tv_sec - CITRUSLEAF_EPOCH );
```

**What is a Write-master?**

When data is written to the cluster, it will first be written to the master node for that record. This is referred to as a “write-master.” Statistics on these writes are available and are different than writes to a node for a replica copy. These replica writes are known as a “write-prole” (see “write-prole” below). Write-masters are used to determine how many unique records have been written to the cluster.

**What is Scan? How does Scan work?**

The scan functionality handles data on a partition-by-partition basis. During a scan, if a created record is in a partition that has already been processed, then it will not be picked up. If the created record belongs to a partition that hasn’t been processed, it will be picked up at the time the partition is processed. By default, scan is throttled back to ensure there is no performance degradation on the front-facing read/writes.

**What does the Expiration Date of a record returned from a scan callback mean?**

The expiration date field returned from a scan is the actual expiry time (Citrusleaf epoch time) for that record, which is computed from the record’s default TTL value (60 days) and the time the record has been written to the database.

**What is the difference between a write-master and a write-prole?**

After a record has been written to the master for that record (see “write-master” above), there will be subsequent writes to the nodes that have the replica. These are known as “write-proles”. If the replication factor has been set at “1″, there will be no write-proles.

**What is the Fire-Forget feature?**

For every write before responding to the client, Aerospike will wait for the prole reply (replica write). You can enable the fire-forget feature, which will return to the client as soon as the master write is successful.
 
<a name="general">
### General Information
</a>

**On what Open Source product is Aerospike based?**

None. Aerospike was not developed on the basis of any Open Source project. It was developed from the ground up to be a highly scalable, low-latency Enterprise-class distributed database. With the acquisition of AlchemyDB, we have very carefully enhanced Aerospike using select technology and more important, expertise from AlchemyDB.


**In what programming language is Aerospike written?**

Aerospike is written in C. This was a choice made for performance and predictability. Aerospike strictly controls the use of memory and does not suffer from garbage collection issues common to products written in other languages such as Java.


**How do I decide how to separate data into namespaces and sets?**

In general, you can think of a “set” in Aerospike as you would a “table” in a relational database. You can also think of an Aerospike “namespace” as a “tablespace” in a relational database.

The best way to start is to understand the sets you will need. Typically, sets will have users, URLs, or similar things as each record in them. Sets that have similar requirements often belong to the same namespace.

Note that since Aerospike does not have a set schema, even sets that are completely different can exist in the same namespace. For example, a namespace may contain sets for people, servers, and URLs. The bins from one namespace do not need to exist in the other namespaces. Conversely, you may also find that even similar items may need to be placed in different namespaces because they may have different needs for how the data will be synchronized between different data centers. For example, you may want a set of users in one namespace that is synchronized across the US and a set of users in a different namespace that is synchronized across Asia.

**Do I need a trial key for Aerospike Community Edition?**

No. Aerospike has chosen to open source the Community Edition. There is no trial key. Access to Enterprise Edition is reserved for those that have entered into a contract with Aerospike.

**Can I store data in RAM?**

Yes. Although Aerospike has technology to make optimal use of flash (SSDs), some customers have chosen to use RAM as storage. It is also possible to have data in one namespace go to RAM while data in another namespace goes into flash all within the same cluster.

**Can I store data on hard disk rather than SSD?**

No. The Aerospike database is intended to be a high performance, low-latency database. Because of this, the physical limitations of rotational disks add an unacceptable amount of latency to the data.

**What are the minimum hardware requirements to run Aerospike?**

Please refer to the System Requirements on the [Plan page](/docs/operations/plan).

**What languages does Aerospike support?**

Please see the [Develop page](/download/client). If you have a language requirement not listed above, please contact us.

**Can I run multiple instances of the Aerospike server on one machine?**

Aerospike does not recommend you do this in production. But if you absolutely need to do this for test or development reasons, there is a way – contact Aerospike for instructions.

**I am currently using the Community Edition of Aerospike.  My application is doing very well and I wish to upgrade the Enterprise Edition.  Is there an upgrade path?  Will downtime be required during the upgrade?**

Aerospike created the Community Edition to enable companies early in their product development cycle to use our platform for free, then seamlessly transition to a full commercial license when needed. The normal path (as done by our current customers) to handle the upgrade is on a rolling basis, one node at a time, without the need for a downtime. This allows you to upgrade your software or hardware while keeping your service up.  See [Upgrade/Repair Server](/docs/operations/upgrade).

**Can we use both Community Edition (CE) and Enterprise Edition (EE)?**

No, you cannot use both CE and EE for Support Reasons. Hybrid Customers will determine the appropriate version for their needs and Aerospike will support the Customer’s efforts to migrate or upgrade as appropriate.


<a name="hardware">
### Hardware
</a>

**How do I load balance across the nodes in a cluster?**

Aerospike automatically distributes both data and traffic to all the nodes in a cluster. There is no need to add additional load balancing. In fact, load balancers tend to cause confusion and reduce performance.

**Can I have mixed hardware configurations for nodes?**

Yes. Aerospike does not require that each node be the same hardware. However, the cluster randomly and evenly distributes data across the nodes, so the cluster will be limited to the performance of the node with the least capacity.

**Can I put more than one Aerospike node on a host?**

Yes. It is possible to put more than one Aerospike node on a host. However, Aerospike makes the most of each host, so there is no need to do this in production. The only reason to put more than one node on a host is for testing/development reasons.

**What is the range of values allowed for the replication factor?**

The minimum replication factor is 1 (no replication). The upper limit is the number of nodes (a copy of the data in each node).

**After a power outage, should all the nodes be started back up all at the same time?**

It is best to bring each node back up one at a time with a 5-10 minute gap.

**Let’s say the power supply on a node dies and I replace it. When the node starts up, does it have to get data from the other nodes or does it read from disk?**

You can actually choose. This is a configurable setting. Depending on the reason for a node crash, you may find that one option or the other is better. For instance, if the problem was a bad disk, you may no longer have the data and will need to get a copy from the rest of the cluster. If the problem was a bad power supply, the data is still on disk and it is faster to read the data from disk than to transfer it from the other nodes.

**If I need to make a configuration change, do I have to restart the server?**

No. Generally you can change the configuration of a node dynamically. This involves issuing some commands for the command line. See the [Configuration page](/docs/reference/configuration) for more details. Changes to most parameters (even for memory settings) can be made active while the server is still up. Changes to the configuration file will not be read dynamically. If a server gets restarted, the server will get its configuration from the configuration file, so any changes that are intended to be permanent should get reflected in the configuration file.

In addition, Enterprise Edition Customers may make changes from the [Aerospike Monitoring Console](/docs/amc/user_guide/enterprise/edit_config.html).

**How do I conduct backups of the database? Will this impact performance?**

Aerospike has a [backup program](/docs/tools/backup) that will gather data from all the nodes and put them into files. This can be done while the cluster is up and serving requests. The machine with the backup does not need to be a node within the cluster, only have network access to it. The backup process is configurable and Aerospike has recommendations to ensure that the backup not affect normal transactions. Most customers can take backups within a few hours with these settings. The backup system is made to run at a lower priority than the front end service, so can take varying amounts of time.

**When a node goes down, how do you reconfigure the system to reroute traffic?**

You don’t. With Aerospike, the cluster will automatically detect when a node has left the cluster. It will then automatically respond by rebalancing data and changing the configuration so that the clients know how to communicate with the cluster.

<a name="database">
### Database
</a>

**How do I define a schema in Aerospike?**

There is no need to define a schema with Aerospike. Every row can have a different set of bins (like columns in a relational database). In fact even the same bin in one record (or row) does not have to have the same data type as the same bin in another record. This flexibility allows developers much freedom to create applications without the normal limitations inherit in a relations database.

**What query language do you support?**

Aerospike has its own API to handle requests to the database. These requests are in the form of get/put/updates to the database, and also include atomic operations on data types such as [Integer](/docs/guide/data-types.html), Double, [List](/docs/guide/cdt-list.html), [Map](/docs/guide/cdt-map.html) and [Bytes](/docs/guide/bitwise.html). Language specific [clients](/docs/client/index.html) implement the API for C, Java, C#, Python, Go, Node.js, PHP, Ruby and others.

**How does Aerospike distribute data/traffic?**

One of the most important aspects in achieving optimal, predictable performance across a cluster is how data and traffic are distributed. Aerospike uses a very random hash (the RIPEMD 160) to make sure that both data volume and traffic are evenly distributed. This occurs automatically and without the need for manual intervention.

With other solutions, you must manually redistribute data.

**Do you support batch gets and puts?**

Aerospike supports batch gets (reads). To avoid confusion on atomicity, it does not currently support batch puts (writes)

**I can’t find an API call to get the key for a given row. How do I get it?**

Aerospike by default does not store the actual key, but rather a one-way hash of the key. What this means is that if you wish to retrieve the key, you should store it as a value. The Java API has a special flag to do this automatically. Make sure you are on a version released after June, 2014.

**If we are using multicast mode for heartbeats, is it recommended to use IGMP snooping to protect the network from broadcast storms and multicast traffic on a VLAN?**

You may choose to either turn off IGMP snooping or to leave it on and turn on the querrier. Failure to do so may result in a cluster than cannot form or will form initally, but dissolve after a few minutes.

**What is Aerospike Available Percent?**

Available Percent is the amount of storage defragmented and the percent available for writing to Aerospike, as a percentage of total space on disk. It is not free disk space. It is the available storage for streaming writes on that particular Aerospike namespace.

**If a node goes down during a read or write, what happens?**

It is possible that a cluster node will go down. The
cluster will identify that this node is no longer sending heartbeats, then form a new
cluster, which will have a new partition map.

The clients regularly check for state changes in the Aerospike Server through a cluster
tending thread. On the next tend interval after the new cluster is formed
(default of 1 second) the clients will learn of the new partition map. Until
then, the clients may try to perform some operations against a node that isn't there.

If a client tries to read from such a node, the operation will time out. When this happens, the client will automatically try reading from a node holding a replica of the record. From a coding standpoint, you do not need to be aware of this, as the API handles the additional attempts at communicating with the database.

If the client tries to write to such a node, the operation will time out. What
happens now depends on write policies given to the client. The clients can be
told to retry the operation a specified number of times with a given interval
between them. The application may choose to catch the timeout and either retry, defer,
or ignore.

As of Aerospike Server version 4.3.0, if a node is being taken down for
maintenance, such as a rolling upgrade, the node should first be [quiesced](/docs/operations/manage/cluster_mng/quiescing_node/index.html).
This will prevent read and write timeouts for the application.  Quiescence is an
enterprise edition feature.

**Is there a way to delete all content from a namespace?**

As of Aerospike Server version 3.12.0, released in March 2017, it is possible to use `truncate` to delete all content from a namespace.

In Aerospike Server versions 4.3.1.11 (and higher 4.3.1.X), 4.4.0.11 (and higher 4.4.0.X), 4.5.0.6 (and higher 4.5.0.X) and 4.5.1.5 and higher, there have been updates made to the `truncate` info command.<BR>
Now `truncate` can delete every record in a given set and `truncate-namespace` can delete every record in a namespace.

In the Enterprise Edition, truncation is durable and preserves record deletions through a cold-restart.<BR>
In the Community Edition, similar to record deletes, records in previously truncated sets are not durable and deletes can return through a cold-start.

More information on understanding the `truncate` command can be found in [Info Command Reference - Truncate](/docs/reference/info#truncate).<BR>
More information on understanding the `truncate-namespace` command can be found in [Info Command Reference - Truncate Namespace](/docs/reference/info#truncate).

<a name="storage">
### Storage
</a>

**Have you tested any SSDs? Which ones do you recommend?**

Aerospike has tested many SSDs and has even created a tool to test the performance of the drives in real world conditions. What is important to realize is that although some SSDs will perform well for short periods of time, Aerospike has found that some will only run into problems after 11 or more hours of use. In order to pass Aerospike’s strict requirements the SSDs must show good performance over an extended period of time. For more information, see the [flash/SSD certification guide](/docs/operations/plan/ssd/ssd_certification.html)

**Does Aerospike require the use of the TRIM command for flash/SSDs?**

No. When Aerospike uses uses a filesystem (optional for data, mandatory for indexes in All-Flash configuration), the filesystem takes care of block management, and when using raw devices, Aerospike controls the device directly, similar to how many relational databases use raw rotational disk. Because of the difference in how they operate, Aerospike has optimized use of SSDs as a NAND device. These optimizations include functionality similar to the TRIM command. The net effect is that performance is improved and garbage collection is distributed throughout the day, while also getting much improved longevity.

**Will performance improve if I use 10 Gb ethernet rather than 1 Gb ethernet?**

While 10 Gb Ethernet obviously has much higher bandwidth, it also has much higher capacity in dealing with transaction throughput. If you need to get more than 100,000 transactions per second out of a single node, then you may want to consider a 10 Gb network. If you don't need such high transaction rates and your network bandwidth requirements fit within 1 Gb, there is no reason to move to a 10 Gb network.


**How do I calculate the amount of space needed in RAM and/or flash (SSD)?**

If you would like an exact algorithm for calculating the amount of space needed, please see the [Capacity Planning page](/docs/operations/plan/capacity). 

If you only need an estimate, Aerospike has a spreadsheet that can assist you with the calculation. Please contact us if you would like a copy and instructions for use.

**Why did you choose the RIPE-MD 160 algorithm for your hash?**

Aerospike chose the RIPE-MD 160 algorithm to hash data for 2 reasons:

- It will take an arbitrary length key and create a consistent 20 byte string.
- The generated string is highly random with an extremely low collision rate.

**How do you read and write data from an Aerospike cluster?**

Every database has its own mechanism for reading or writing. The exact method will directly impact the performance of these operations. Aerospike has optimized for both operations to occur on the cluster at the same time. Regardless as to whether you are reading or writing to the cluster, your code will not need to worry about the state of the cluster. All the work will be handled by the Aerospike intelligent API. It will keep track of which node to connect to for the operations.

- **Write**
When you issue a write command for a key/value pair, the Aerospike API will hash the key and use its value to determine which node is the master (primary) for that key. It will then connect to the node and commit the value to memory. If the replication factor is 2 or greater, the node will then communicate with the other nodes to commit the data in memory. When these have all been committed in memory, the node returns the client as committed. If using SSDs rather than RAM to store the data, the writes will be streamed to disk for persistence.

- **Read**
When you issue a read command for a key, the Aerospike API will again hash the key and use the value to determine which node is the master for that key. It will then connect to it to get the value. No other connection is necessary, even if there are replicated copies in the database. If the master is down, the API will automatically reroute the request to a replicated copy. No changes to the client code or configuration are needed.


**I have used databases in the past that reclaim space using a process called “compaction.”  This process takes tremendous resources and sometimes results in instability.  How does Aerospike handle reclamation of space?**

Aerospike was designed from the beginning as an enterprise-class database that would stay up 24/7. Rather than writing large files and compacting them at one time, a process reclaims space constantly throughout the day. This leads to much greater predictability and reliability.

  How this works on flash (SSD):

Storage is split into two areas: index (stored either in RAM or on flash) and data (stored on flash).
When data is written to a node, an entry is made in the index and the data is streamed to the flash in blocks.  These writes are intended to take full advantage of how flash writes.

At some point, a delete or update may occur to a record.  This means that the entry in the index will get either deleted or updated to point to a different location on the flash device. This may seem similar to a compaction, but it is not. Since Aerospike does not use a filesystem to store records, we do not use “compaction.”  Rather, there is a separate job that goes through the flash device and reclaims the space in what we refer to as “defragmentation.” This process runs constantly throughout the day (every few seconds by default).  What this means is that you will not get a single big event to reclaim space, but many very small ones.

In addition to the above, there is another job that goes through the index and expires data that has aged beyond its configureable time-to-live (TTL). You can alter the time-to-live value in the configuration file or override it in your client code.  This process effectively deletes the index so that the space on flash device can be reclaimed by the defragmentation process.

**How do deletes work?**

Delete operations only delete the index entries (whether memory or flash) for the records. The actual records on the disk are asynchronously removed by a separate defragmentation process which then enables new records to overwrite old ones in defragmented blocks. With defragmentation, the database reclaims storage that is no longer needed. In Aerospike there is no need to explicitly mark records for deletion, thereby reducing writes. This deletion approach is extremely effective in case of flash devices, which may have limited write cycles.

However, with the above performance gains that Aerospike achieves with index-based deletion, there is a small chance that the deleted data may not actually be deleted. Consider a scenario where a delete is issued. Before the asynchronous process deletes the record from the disk and new records overwrite it, the node itself is rebooted. If there is a cold start reboot, the deleted data will be re-indexed. 

There are some best practices to avoid this scenario. Starting with release Enterprise Edition 2.1.2, with a Fast Restart, Aerospike does not rebuild the whole index from disk on reboot. This will make sure that the deleted data will not get restored to the index. Additionally, instead of a delete, we recommend that you set a TTL for all the records that go into the database. Records whose TTL have expired will never be reindexed after a reboot. Finally, version 3.10 introduces the durable delete policy, creating tombstones to prevent older versions of a record to reappear. See details on the [durable delete](/docs/guide/durable_deletes.html) page.

**As part of XDR, will deletes be shipped to the destination?**

Starting with Aerospike release 2.1.3, XDR ships deletes to the remote destination.

<a name="data">
### Data Migration and Synchronization
</a>

**How long does a migration take?**

Migrations do not take a set amount of time. You can configure the amount of dedicated resources. Aerospike has default settings that will ensure that if a migration occurs, it will not impact the performance of the cluster.

You will not know when the migration will be completed because of varias variables: load on system, cluster size, migration tables, etc. But there are ways we can extrapolate to see how much migration reduces over a period of time and then estimate it, (as long as cluster state do not change and nodes are not restarted, which would then reset everything).

Migrations can be controlled by the following config parameters: `migrate-xmit-hwm`, `migrate-xmit-lwm`. Also a static value `migrate-threads` which we recommend at 1 and if changed, requires node restart.

The way these parameters interact is whenever queue is greater than `migrate-xmit-hwm` it will stop migrating data. Whenever it goes below `migrate-xmit-lwm` it will start migrating data.

Times will vary depending on a few different factors, including the network and amount of data per node. For typical loads, expect a migration to take one to several hours.

**How does Aerospike handle data conflicts?**

By default, Aerospike uses a generation counter to determine which of 2 values for a key is the correct one. It will choose the value that has a higher generation count.

**If I am synchronizing 2 datacenters using the Aerospike XDR product (available in the Enterprise Edition), what happens if the network connection is severed? Will I lose data or will it be stored somewhere?**

You will not lose data. Aerospike XDR is configured with a log file that stores the records to transfer to the other data center. In the event that the connection is down, the log will increase. Aerospike XDR only stores a hash of the key for each record rather than the entire record in order to conserve space. The amount of space allocated is configurable and can be many days long. Aerospike does not have high performance demands on access to this file, so you may choose to use standard rotation hard disk.

**When synchronizing data between data centers using Aerospike XDR, how is the data transfered? Do the same number of nodes need to be in each data center?**

When using Aerospike XDR, the simplified view is that there are 2 clusters in 2 data centers, the local and the remote. Each node in the local data center acts as a client to the remote data center. As a result the configuration of the two clusters do not have to be the same. They may have different numbers of nodes and different hardware.
