---
title: Primary Index
description: In Aerospike, the primary key index provides predictable and fast access to database row information.
assets: /docs/architecture/assets
---

In an operational database, the fastest and most predictable index is the primary key index. In Aerospike, the primary key index is a blend of distributed hash table technology with a distributed tree structure in each server. The entire keyspace in the namespace (database) is partitioned using a robust hash function into *partitions*. A total of 4096 partitions are equally distributed across cluster nodes. See [data-distribution](/docs/architecture/data-distribution.html) for details on hashing and partitioning.

At the lowest level, Aerospike uses a red-black in-memory structure. For each partition, there can be configurable number of such red-black structures, termed sprigs.  Configuring the right number of sprigs on a machine trades-off memory overhead while optimizes parallel access.

{{#todo}}
```bash
            RIPEMD160 Hash         
Primary Key -------+ 
                   |           Partition hash
                   +--> digest ----------+
                                         |
                                        \|/ 
                                       +-----+
                                       |  1  |------------> 
                                       |     |            /\
                                       |     |           /  \
                                       |     |          / primary key
                                       |     |         / r-b tree
                                       |     |        /________\
                                       |     |
                                       |     |
                                       |     |
                                       |     |------------> 
                                       |     |            /\
                                       |     |           /  \
                                       |     |          / primary key
                                       |     |         / r-b tree
                                       |     |        /________\
                                       |4096 | 
                                       +-----+
```
{{/todo}}

The primary index is on the 20 byte hash&mdash;the *digest* of the specified primary key. While this expands the key size of some records (for example, an integer key which is only 8-bytes), its is beneficial because code operation is predictable regardless of input key size or distribution.

When a single server fails, the indexes on a second server are immediately available. If the failed server remains down, data starts rebalancing, and replicated indexes builds up on new nodes.

### Index Metadata

Currently, each index entry requires 64 bytes. In addition to the 20-byte digest, the following metadata are also stored in index.

- **write generation:** Tracks all updates to the key; used for resolving conflicting updates.

- **expiration time:** Tracks time when a key expires. The eviction subsystem uses this metadata.

- **last update time:** Tracks the last writes to the key (Citrusleaf epoch). Used for conflict resolution during cold restart, conflict resolution during migration (based on config), predicate filtering, incremental backup scans, truncate and truncate-namespace commands.

- **storage address:** The  storage location (both in-memory and persistent) for data.

### Index Persistence

To maintain throughput, primary indexes are not committed to storage&mdash;only to RAM. This allows high performance, highly parallel writes. The data storage layer can be configured not to use storage. When the Aerospike server starts, it walks through data on storage and creates a primary index for all partitions.

#### Fast Restart Feature

For fast cluster upgrades with minimal downtime, Aerospike supports **fast restarts**.  The fast restart feature allocates index memory from a Linux shared memory segment. For planned shutdowns and restarts of Aerospike (for example, for a upgrade), on restart the server simply re-attaches to the shared memory segment, and activates the primary indexes without a data scan of the storage.

### Single Bin Optimization

Enabling the Aerospike **single bin** feature on a namespace provides lower memory usage than when supporting the Aerospike bin structure on each record. A further optimization is achieved when all the values in an in-memory, single-bin namespace, are integer or double, and the namespace is declared **data in index**. The saved space in the primary index is then reused to store the integer or double value. This means that the amount of storage required for the namespace is simply the space required for its primary index.

