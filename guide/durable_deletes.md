---
title: Durable Deletes
description: Aerospike Durable Deletes policy. 
assets: /docs/guide/assets
---

By default, Aerospike deletes data in a very efficient way, reclaiming the memory instantaneously by only dropping the primary index entry for the deleted object. This has the disadvantage of having previously persisted versions of such deleted objects potentially resurrected upon cold starting the namespace. In order to address this, Aerospike introduced Durable Delete in Enterprise Edition version 3.10. 

Durable Delete can be specified as a policy on a transaction basis. The durable delete policy can be supplied on the following calls:

+ Write (applicable only when the last bin is removed, resulting in record delete)
+ Delete
+ Operate (applicable only when the last bin is removed, resulting in record delete)
+ UDF (applicable only when UDF execution results in a record delete, either through delete call, or last bin removal)


### Durable Delete and Tombstone

When a durable delete is issued a tombstone is written. The tombstone-write is similar to a record-update in that:

- It continues to occupy entry in the index, together with other record entries in index.
- It is persisted on disk, together with previous copies of the record on disk.
- It has the same meta-data as any other record:
    - last-update-time – just like normal update.
    - generation – increments just like normal update.
- It is replicated at the same replication factor specified on the namespace.
- It is migrated the same way current records are migrated.
- It is conflict resolved the same way as data-records.

Tombstones are written as no-expiration.

As Durable Delete depends on accurate clocks, it is important to have clocks in sync across the cluster.


### Tombstone on Cold Start

A Tombstone is simply a record without any bins.

- It contains all meta data, including key.
- It does not contain LDT flag.

On a cold start, disks are scanned to rebuild the in-memory index tree for the records. Version of the records are compared. The version with the most recent last-update-time (with tiebreak using record generation) is brought back. For a record that is durably deleted, the tombstone is just a version that participates in the comparison, and can prevent any older versions of the record from returning. If a tombstone is the most recent version, it will be reloaded into index.

### Tombstone Management

Similar to data records, tombstones are also reclaimed when they are removed. When removed, they are removed from the in-memory index, and the on-disk copy is eligible for de-fragmentation. Index memory is immediately re-usable. The storage is re-usable based on when the space is defragmented.

A special background mechanism ("Tomb-Raider") is used to remove no-longer needed tombstones. The conditions for a tombstone to be removed are as follows:

- There are no previous copies of the record on disk.
    - This condition assures that a cold start will not bring back any older copy.
- The tombstone's last-update-time is before the current time minus the configured [`tomb-raider-eligible-age`](/docs/reference/configuration/#tomb-raider-eligible-age).
    - This condition prevents a node that's been apart from the cluster for `tomb-raider-eligible-age` seconds, to rejoin and re-introduce an older copy.
- The node is not waiting for any incoming migration.

If all conditions are satisfied, the tombstone will be reclaimed.

The actual background thread is split into roughly the following steps:

- Iterating through index to mark all tombstones as candidates for removal (cenotaphs).
- Scan each disk block for records, un-mark cenotaph for each record.
- Iterate through index again. All cenotaphs remaining are candidates for permanent removal.

For non-persisted namespace, tombstone removal is separate and only requires one index iteration for tombstone removal.

Cold start also removes unneeded tombstones as part of the disk reading:

- All tombstones are marked as candidates for removal (cenotaphs) on initial bring-up.
- If a subsequent live record which the tombstone covers is read, cenotaph will be unmarked, and tombstone stays.
- Otherwise, at end of cold start, all cenotaphs will be deleted.

The following configurations are available to control the behaviour of the Tomb-raider: 

- [`tomb-raider-period`](/docs/reference/configuration/#tomb-raider-period) - minimum amount of time, in seconds, in between runs, default is 1 day (86400).
- [`tomb-raider-eligible-age`](/docs/reference/configuration/#tomb-raider-eligible-age) - number of seconds to retain a tombstone, even though it's discovered to be safe to remove, default is 1 day (86400).
- [`tomb-raider-sleep`](/docs/reference/configuration/#tomb-raider-sleep) (storage-only) - number of micro-seconds to sleep in between large block reads on disk, default is 1000 µs (1 ms).


### Expired and Evicted Records

Expired and evicted records do not generate tombstones. This is desirable behavior:

- It allows maximum resource capacity to be used for data records instead of tombstones.
- If resource capacity increases, for example, by increasing the memory capacity on a node, it is possible to revive the non durable deleted record on cold-start. 

There are also other conditions where evicted records may return:

- A replica with a shallower cold-start eviction-time than the master. In this case, when the master node departs, and replica is cold started, the replica records may revive.

Tombstones never expire, thus not eligible to eviction.


### XDR

When a durable delete is executed on a source cluster,  it is also asynchronously replicated to remote data-center as a durable delete. (Assuming the [`xdr-delete-shipping-enabled`](/docs/reference/configuration#xdr-delete-shipping-enabled) is set to true).


### Scan, Batch

Scan/Batch will skip returning tombstoned records.


### Conflict Resolution Policy

The conflict resolution policy affects durable delete behavior across cluster state changes. To guarantee correct propagation of durable deletes, the [`conflict-resolution-policy`](/docs/reference/configuration#conflict-resolution-policy) should be "last-update-time".


### Caveats

#### Capacity Sizing, Tuning Requirement

The key caveat to be aware of when switching on Durable Delete is the impact on cluster sizing.  Previously clusters have been sized according to the number of active records within the cluster.  Tombstones take up RAM and disk space, they never expire and thus can never be evicted.  This is reported in logs and statistics. **It is advisable to open a use-case and sizing discussion with Aerospike Solutions Architects to discuss use cases and how use of Durable Delete could affect cluster sizing**. The sizing impact is as follows:

+ Index space = 64 bytes + optional key size
+ Disk space = 128 bytes + optional key size

Another potential impact of Durable Delete is an increase in Large Block Reads on the SSD as the Tomb Raider sweeps the disk.  This could, if not correctly sized, introduce latency into the cluster which could impact transactional operations.


#### Tombstone Reporting

- Log lines have been updated to report the status of tombstones within the namespace on the node:

```
Aug 24 2016 22:13:50 GMT: INFO (info): (ticker.c:336) {test} objects: all 4344 master 2209 prole 2135
Aug 24 2016 22:13:50 GMT: INFO (info): (ticker.c:372) {test} tombstones: all 2378 master 1293 prole 1085
```

- Additionally the following namespace statistics are available to track tombstones:

```
$ asinfo -v "namespace/test" -l | grep tomb
tombstones=2378
master_tombstones=1293
prole_tombstones=1085
tomb-raider-eligible-age=86400
tomb-raider-period=10000
```


#### Client Server Compatibility

- Default client policy is NOT durable delete to keep backward compatibility.
- Client applications must be enhanced to use durable-delete feature.
- New client with durable delete policy writes against an older version of server will simply ignore the durable delete policy.


#### Community/Enterprise Compatibility

If a drive for Enterprise and has tombstone, downgrades to Community, cold-start will fail when reading the tombstone. Drives will need to be cleaned up for successful restart.

One new error code is introduced:

`AS_PROTO_RESULT_FAIL_ENTERPRISE_ONLY` - if a durable delete policy is issued against a Community Edition server.
