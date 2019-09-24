---
title: Cold Start
description: Learn how to manage a Cold Restart
---

## When does Aerospike cold restart?
After a shutdown, Aerospike will rebuild the primary index, which
[Enterprise Edition](/products/product-matrix) stores in shared memory, in the
following situations:

- [Community Edition](/products/product-matrix) always cold starts since the
  [fast-start](/docs/operations/manage/aerospike/fast_start/) feature is
  exclusive to the Enterprise Edition.
- After stopping unexpectedly (for example a segmentation fault, out of memory
  situation, or a kernel freeze).
- After a server reboot (for example due to a kernel upgrade or RAM addition).
- For server versions prior to 3.15.1.3, if namespace is configured for
  [`data-in-memory true`](/docs/reference/configuration/#data-in-memory).
- Cold restart is forced using the 'coldstart' command-line option.
- If applicable, as part of an expected upgrade path - e.g. upgrading to version
  [4.2.0.2](/download/server/notes.html#4.2.0.2).

See ['When does fast restart NOT happen?'](/docs/operations/manage/aerospike/fast_start#when-does-fast-restart-not-happen-) for more details.

## What are the impacts of cold restart?
After a cold restart, the index is repopulated from storage. There can be
multiple copies of the data existing on the device/file depending on the
frequency of updates and if the blocks holding the older data have not yet been
defragmented and subsequently overwritten with new data. Here are some
consequences of a cold restart:

- Aerospike daemon start up takes much longer (compared to a
 [fast restart](/docs/operations/manage/aerospike/fast_start/index.html) -
 Enterprise Edition only). This can be exacerbated if evictions are triggered
 during the cold restart. 
- [Non-durably deleted](/docs/guide/durable_deletes.html) records may be
  reverted to an older version.
- When running [`strong-consistency`](/docs/reference/configuration/#strong-consistency)
  enabled namespaces, cold restarts has potential further impacts:
  - All records will be marked as **unreplicated** (refer to the
    [`appeals_tx_remaining`](/docs/reference/metrics/?show-removed=1#appeals_tx_remaining)
    stat).
    - Unreplicated records on a single node should not have a significant impact
      as they should be quickly resolved (checking against the current master)
      prior to migrations starting. 
    - In case of multiple nodes being cold restarted, partitions having all
      their roster replicas on those nodes will incur additional latency as it
      would require initial transaction to an unreplicated record to
      [re-replicate](/docs/reference/metrics/#re_repl_success) the record.
  - When recovering from an ungraceful shutdown (power loss for example),
    partitions will be marked as un-trusted, excluding them from being counted
    as a valid replica in the roster (will not count toward a super majority for
    example).
    - This would degrade overall availability in case of subsequent network
      partitions (until migrations complete).
    - In case of multiple nodes cold restarting following an ungraceful
      shutdown, partitions having all their roster replicas on those nodes will
      be marked as dead upon re-formation of complete roster (refer to the
      [`dead_partitions`](/docs/reference/metrics/#dead_partitions) metric).

## Different use cases to consider
In order to clearly understand the consequences, you would need to quantify the
type of data and the use-case for your application on your servers. 

The following details can be applied on a per namespace basis depending on their
particular usage.

**It is always recommended to [backup](/docs/tools/backup/asbackup.html) the
data on the cluster prior to maintenance, especially when a cold restart would
be triggered.**

### 1) Data expiring naturally without ever having any record's expiration time shortened or durably deleted.

This is the best case scenario. Here are the exact conditions required: 

- Records are exclusively deleted using the durable delete policy.
- Records with ttl set never have their expiration time shortened. 
- Records have never been evicted.

In this situation, a cold restart will reload the same data that existed
previously. Durably deleted or expired data will not be resurrected.

### 2) Data explicitly non-durably deleted, evicted, or records having their expiration time shortened.

Here are the conditions for this scenario:

- Records have been deleted without the durable delete policy.
- In Community Edition, records have been deleted through the
  [`truncate`](/docs/reference/info/index.html#truncate) or [`truncate-namespace`](/docs/reference/info/index.html#truncate-namespace) info commands. Note that truncation  
  is durable in the Enterprise Edition.
- Records with a ttl set have had their expiration time shortened.

Here are ways to handle a cold restart without bringing back previously deleted
records.

As stated previously on this page, **it is always recommended to back up the
data on the cluster prior to maintenance, especially when a cold restart would
be triggered.**

#### a) Manually clean up the node's persistent storage before starting it back up. 

{{#warn}}Must have a replication factor 2 or more on the namespace.{{/warn}}

1. Clean up the persistence storage:
    - If using a file for persistence, simply delete the file. 
    - If using a raw device, use the
    [`dd` command](/docs/operations/plan/ssd/ssd_init.html#erasing-the-drive) or
    [`blkdiscard` command](https://discuss.aerospike.com/t/zeroize-multiple-ssds-simultaneously/716).
2. Introduce the empty node into the cluster.
3. **Wait for migrations to complete** (which will fully repopulate the data on
   this node) before proceeding with the next node.

#### b) Use the [`cold-start-empty`](/docs/reference/configuration#cold-start-empty) configuration

{{#warn}}Must have a replication factor 2 or more on the namespace.{{/warn}}

This configuration will instruct the server to ignore the data on disk upon cold
restart. Migrations will then rebalance the data across the cluster and
repopulate this node. It is therefore necessary to **wait for the completion of
migrations** before proceeding with the next node. Once a node has been
restarted with `cold-start-empty true`, it is typically not recommended to
remove this configuration without a full 'cleaning' the persistent storage as
described in (a).

Note that although the data on the persistent storage is ignored, it is not
removed from the device. Therefore, a subsequent cold restart would potentially
resurrect older records (if the `cold-start-empty` configuration is set to
false). Setting the `cold-start-empty` to false would prevent data
unavailability (and potential data loss) in the event of multiple node cold
restarting at close interval (before migrations complete).


#### c) Cold restart the node 'as is'

If the use case can handle the potential resurrection of deleted records, the
node can be cold restarted without any further action.

{{#note}}
As deleted records may be resurrected, much more data than previously existing
may be loaded.
{{/note}}

{{#note}}
In the event of a cold restart causing evictions to occur (disk or memory high
water mark breached), the start up time could be drastically increased. Refer to
the following article for details on
[Speeding up cold restart evictions](https://discuss.aerospike.com/t/faq-what-options-are-available-to-speed-up-cold-start-eviction/3480).
{{/note}}

{{#warn}}
If evictions are not effective during the cold restart (for example, records do
not have any ttl set) the `stop-writes-pct` threshold could be breached. In such
event, the node will abort and not complete the start up.
{{/warn}}

