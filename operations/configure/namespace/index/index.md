---
title: Configuring The Primary Index
description: Configuring The Primary Index
---

Aerospike's primary index may be stored in three different ways: RAM, persistent memory (PM), and flash (normally NVMe SSDs).
Different namespaces within the same cluster may use different index storage methods.

To specify an index storage method, use the namespace context configuration item [`index-type`](/docs/reference/configuration/#index-type).

The default [`index-type`](/docs/reference/configuration/#index-type) is `shmem`, with the index stored in RAM (Linux shared memory).
To specify a persistent memory index, use [`index-type`](/docs/reference/configuration/#index-type) `pmem`, and to specify an index in flash, 
use [`index-type`](/docs/reference/configuration/#index-type) `flash`.

{{#warn}}

 In a [Systemd environment](https://discuss.aerospike.com/t/the-asd-process-cold-starts-after-a-dirty-exit-which-appears-to-be-clean/6472) you may need to increase __TimeoutSec__ from the default of 15s. This setting is located in /usr/lib/systemd/system/aerospike.service. This will prevent systemd from killing the asd process prematurely while the service is being shutdown. The primary index clean-up process during shutdown may take longer than the default 15s. (This default value has been increased to 10 minutes as of version [4.6.0.2](https://www.aerospike.com/enterprise/download/server/notes.html#4.6.0.2) )
{{/warn}}

## Persistent Memory (PM) Index

Aerospike's PM index feature allows primary indexes to be stored in
persistent memory - e.g., Intel Optane DC NVDIMMs - instead of
RAM-based shared memory segments.

Unlike a RAM-based index, a PM index is preserved across reboots of
a cluster node's OS, which allows for [fast restarts](/docs/operations/manage/aerospike/fast_start/index.html) of Aerospike after
a reboot.

Aerospike requires the persistent memory to be accessible via _fsdax_,
i.e., via block devices such as _/dev/pmem0_:

  * The NVDIMM regions must have been configured as _AppDirect_
    regions, as in the following example from a machine with a 750-GiB
    _AppDirect_ region:

    ```
$ sudo ipmctl show -region
SocketID             ISetID    PersistentMemoryType  Capacity FreeCapacity HealthState
       0 0x59727f4821b32ccc               AppDirect 750.0 GiB      0.0 GiB     Healthy
    ```

  * The NVDIMM regions must have been turned into _fsdax_ namespaces,
    as in the following example from the same machine:

    ```
$ sudo ndctl list
[
  {
    "dev":"namespace0.0",
    "mode":"fsdax",
    "blockdev":"pmem0",
    ...
  }
]
    ```

The PM block device must contain a file system that is capable of
_DAX (Direct Access)_, such as _EXT4_ or _XFS_. On the machine in the
above example, this could be accomplished in the usual way:

EXT4 filesystem:

```
$ sudo mkfs.ext4 /dev/pmem0
```

or with XFS:

```
$ sudo mkfs.xfs -f -d su=2m,sw=1 /dev/pmem0

```

Finally, the file system needs to be mounted with the _dax_ mount
option. In the following example, we use _/mnt/pmem0_ as the mount
point.

```
$ sudo mount -o dax /dev/pmem0 /mnt/pmem0
```

The _dax_ mount option is very important. Things may seem to work
without it. However, without this option, the Linux page cache would
be involved in all I/O to and from persistent memory, which would
drastically reduce performance.



{{#warn}}

Remember to make the mount persistent to survive system reboots by adding it to /etc/fstab. The mount point config line can be copied from /etc/mtab to /etc/fstab.

{{/warn}}







The primary index type is configured per namespace. To enable a PM
index for a namespace, add an
[`index-type`](/docs/reference/configuration/#index-type) subsection
with an index type of `pmem` to its namespace section. The added
[`index-type`](/docs/reference/configuration/#index-type) subsection must contain:

  * One or more [`mount`](/docs/reference/configuration/#mount)
    directives to indicate the mount points of the persistent memory
    to be used for the PM index.

    A single namespace can use persistent memory across multiple mount
    points and will evenly distribute allocations across all of
    them.

    Conversely, mount points can be shared across multiple
    namespaces. The file names underlying namespaces' persistent
    memory allocations are namespace-specific, which avoids file name
    clashes between namespaces when they share mount points.

  * A
    [`mounts-size-limit`](/docs/reference/configuration/#mounts-size-limit)
    directive to indicate this namespace's share of the space
    available across the given mount points.

    When multiple namespaces share mount points, this configuration
    directive tells Aerospike how much of the total available
    memory across mount points each namespace is expected to use.

    The specified value, along with configuration item
    [`mounts-high-water-pct`](/docs/reference/configuration/#mounts-high-water-pct)
    (default is 80) forms the basis for calculating the high
    watermark for evictions, for example.

    If mount points are not shared between namespaces, then simply
    specify the total available space.

    Ensure __mounts-size-limit__ is lower or equal to the size of the filesystem.

The following configuration snippet extends the above example and
makes all of _/mnt/pmem0_ memory (i.e., 750 GiB) available to the
namespace:

```bash
namespace test {
    ...
    index-type pmem {
        mount /mnt/pmem0
        mounts-size-limit 750G
    }
    ...
}
```

## Flash Index

The Aerospike All Flash feature allows primary indexes to be stored on flash memory devices, typically NVMe SSDs.

This index storage method is typically used for extremely large primary indexes with relatively small records.
Accuracy is critical for certain aspects of capacity planning and configuration.

{{#warn}}
It is important to understand the subtleties of [All Flash Sizing](/docs/operations/plan/capacity/index.html#aerospike-all-flash) as scaling up an All Flash namespace may require an increase of [`partition-tree-sprigs`](/docs/reference/configuration/index.html?show-removed=0#partition-tree-sprigs) which would require a rolling [Cold Restart](/docs/operations/manage/aerospike/cold_start/). 
{{/warn}}

To enable a flash index for a namespace, add an
[`index-type`](/docs/reference/configuration/#index-type) subsection
with an index type of `flash` to its namespace section. The added
[`index-type`](/docs/reference/configuration/#index-type) subsection must contain:

  * One or more [`mount`](/docs/reference/configuration/#mount)
    directives to indicate the mount points on the flash storage
    to be used for the flash index.

    A single namespace can use flash index storage across multiple mount
    points and will evenly distribute allocations across all of
    them.

    Conversely, mount points can be shared across multiple
    namespaces. The file names underlying namespaces' flash index
    allocations are namespace-specific, which avoids file name
    clashes between namespaces when they share mount points.

  * A
    [`mounts-size-limit`](/docs/reference/configuration/#mounts-size-limit)
    directive to indicate this namespace's share of the space
    available across the given mount points.

    When multiple namespaces share mount points, this configuration
    directive tells Aerospike how much of the total available
    space across mount points each namespace is expected to use.

    The specified value, along with configuration item
    [`mounts-high-water-pct`](/docs/reference/configuration/#mounts-high-water-pct)
    (default is 80) forms the basis for calculating the high
    watermark for evictions, for example.

    If mount points are not shared between namespaces, then simply
    specify the total available space.


{{#note}}
An XFS file system is recommended as it seems to provide better concurrent access to files compared to EXT4.
{{/note}}

{{#note}}
Having more physical devices improves performance by increasing parallelism across those. 
More partitions per physical device doesn't necessarily improve performance. 
Aerospike instantiates at least 4 different arena allocations (files) and will allocate more 
if more devices (logical partitions or physical devices) are present. Instantiating more than 1 
arena at a time helps with contention against the same arena, which is important during heavy 
insertion loads.
{{/note}}


Sample configuration snippet:

```bash
namespace test {
    ...
    partition-tree-sprigs 1M # Typically very large for flash index - see sizing guide.
    ...
    index-type flash {
        mount /mnt/nvme0
        mount /mnt/nvme1
        mount /mnt/nvme2
        mount /mnt/nvme3
        mounts-size-limit 1T 
    }
    ...
}
```

### Flash Index Calculations Summary

Refer to the [Sizing Guide](/docs/operations/plan/capacity/index.html#aerospike-all-flash) for details.

Here is a short summary for calculating the disk space and memory required for a 4 billion records namespace with 
a replication factor of 2.

Number of sprigs required:

- 4 billion (records) / 4096 partitions / 32 (records per sprig, to retain half-fill-factor) = ~30,517
- Round up to power of 2: 32,768 sprigs per partition

Disk space required:

- 32,768 (sprigs per partition) * 4096 (partitions) * 2 (replication factor) * 4KiB (size of each block) = 1TiB for the whole cluster
- 1TiB (required for the whole cluster) / 3 (minimal number of nodes) / 0.8 (mount-high-water-pct at 80%) = 427 GiB per node

RAM required:

- 32,768 (sprigs per partition) * 4096 (partitions) * 2 (replication factor) * 13 bytes (memory required per sprig) = 3,328 MiB for the whole cluster
- 3,328 MiB (required for the whole cluster) / 3 (minimal number of nodes) / 0.8 (mount-high-water-pct at 80%) = 1,387 MiB per node

{{#note}}
Since All Flash uses a filesystem with multiple files, the mountpoint size should be slightly larger than 427 GiB, in order to accomodate the filesystem overheads. This is filesystem dependent. The number given, 427 GiB, is for actual usable space storage inside the files.
{{/note}}


