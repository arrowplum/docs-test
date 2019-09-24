
## Memory Required

In Aerospike's [Hybrid Memory](/docs/architecture/storage.html) architecture the indexes are always stored in RAM. There must be sufficient RAM for both the [primary index](/docs/architecture/primary-index.html) and [secondary indexes](/docs/architecture/secondary-index.html). You should provision enough RAM so that you do not exceed the [high water mark](/docs/operations/configure/namespace/retention/index.html), which by default is 60% of the [`memory-size`](/docs/reference/configuration/#memory-size) of the namespace.

Starting with Aerospike Enterprise Edition 4.3, the primary index of a namespace
can be stored on a dedicated device. In this configuration, known as Aerospike All Flash, memory consumption is reduced to a bare minimum.

{{#warn}}
Make sure that the combined `memory-size` of your namespaces does not exceed
the available RAM on the machine. Enough memory should be reserved for the OS,
namespace overhead, and other software running on the machine.
{{/warn}}

### Primary Index

The primary index of each namespace is partitioned into 4096 partitions, and each partition is
structured as a group of [sprigs](https://discuss.aerospike.com/t/faq-what-are-sprigs/4936) (shallow red-black trees).
The sprigs point to record metadata, 64 bytes per record.

#### Aerospike Hybrid Memory
In Aerospike's standard Hybrid Memory configuration, the RAM consumed by the
primary index is:

    64 bytes × (replication factor) × (number of records)

[Replication factor](/docs/architecture/data-distribution.html) is the number of copies each record has within the namespace.
The default replication factor for a namespace is 2 - a master copy and a
single replica copy for each record.

#### Aerospike All Flash

{{#note}}
Refer to the [Primary Index Configuration](/docs/operations/configure/namespace/index/index.html#flash-index) 
page for further details.
{{/note}}

{{#warn}}
It is important to understand the subtleties of All Flash sizing as scaling up an All Flash namespace may require an increase of [`partition-tree-sprigs`](/docs/reference/configuration/index.html?show-removed=0#partition-tree-sprigs) which would require a rolling [Cold Restart](/docs/operations/manage/aerospike/cold_start/). Adding nodes will add more capacity but as sprigs fill up and overflow their initial 4KiB disk allocation, performance would be impacted.
{{/warn}}


When a namespace uses an All Flash configuration, the 64 bytes of record
metadata is not stored in memory, but rather as part of a 4KiB block on an
index device. Only the sprig portion of the primary index consumes RAM,
13 bytes _per-sprig_.

To reduce the number of read operations to the index device, consider the
'fill fraction' of an index block. You should ensure that each sprig contains
fewer than 64 records (as 64 x 64B is 4KiB).

If the namespace is projected to grow rapidly, you will want to use a
lower fill fraction, to leave room for future records. Full sprigs will span more
than a single 4KiB index block, and will likely require more than a single index
device read. Modifying the number of sprigs, to mitigate such a situation,
requires a cold start to rebuild the primary index. So, it's better to determine
the fill factor in advance.

    sprigs per partition = (total unique records / (64 x fill fraction)) / 4096

[`partition-tree-sprigs`](/docs/reference/configuration/#partition-tree-sprigs)
must be a power of 2, so whatever the above calculation yields, pick the
nearest appropriate power of 2.

For example, with 4 billion unique objects, and a fill factor of 1/2, the sprigs
per partition should be:

    (4 x 10^9) / (64 x 0.5) / 4096 = ~ 30,517 -> nearest power of 2 = 32,768

Each sprig requires 13 bytes of RAM overhead so

    total sprigs overhead = 13 bytes x total unique sprigs x replication factor
                          = 13 bytes x ((number of records x replication factor) / (64 x fill fraction))

The total sprigs are then divided evenly over the number of nodes.

For our previous example, the amount of memory consumed by the primary index
sprigs is:

    13 bytes x 32768 x 4096 x 2 = 3.25GiB

So with 4 billion objects and a replication factor of 2, the RAM consumed in
association with the primary index (across the cluster) in All Flash is 3.25GiB,
instead of 476.8GiB of RAM that would be used by the same example in a Hybrid
Memory configuration.

When calculating the number of required sprigs, calculations must also be made to ensure
the correct amount of space is provided on the disk for the Primary Indexes.
The [`mounts-size-limit`](/docs/reference/configuration/#mounts-size-limit) should then
be adjusted. To do so, the following formula is to be applied to get the minimum size
needed for this configuration parameter.

    4096 * replication-factor / min-cluster-size * partition-tree-sprigs * 4KB

To explain the above, the [`mounts-size-limit`](/docs/reference/configuration/#mounts-size-limit) should be 4096 (the number of master partitions), 
multiplied by [`replication-factor`](/docs/reference/configuration/#replication-factor) to get the total number of partitions, 
divided by the [`minimum cluster size`](/docs/reference/configuration/#min-cluster-size) that you will have, to get the partitions per node maximum, 
multiplied by the number of [`partition tree sprigs`](/docs/reference/configuration/#partition-tree-sprigs) to get the max sprigs per node, 
and then multipled by 4KB (as each sprig occupies a minimum of 4KB). This should be the minimum size of the usable mount size for your Primary Indexes.
Please also take into account the file system overhead when partitioning the disk for the All Flash mounts.

### Secondary Indexes

Secondary indexes can be optionally built over your data.
See the [Secondary Index Capacity Planning](/docs/operations/plan/capacity/secondary_indexes.html) for more details.

### For Data in Memory

If a namespace is configured to store data in memory, the RAM requirement for
that storage can be calculated as the sum of:

- Overhead for each record:

    `2 bytes`
- If the [key](/docs/architecture/data-model.html#records) is saved for the record:

    `+ 13 bytes overhead + (8 bytes (integer key) OR length of string/blob (string/blob key))`
- General overhead for each bin:

    `+ 12 bytes`
- Type-dependent overhead for each bin:

    `+ 0 bytes for integer/float data, 5 bytes for string, blob, list/map, geojson data`
- Data: size of data in all the record's bins (0 bytes for integer data, which is stored by replacing some of the general overhead). Please see [Data Size](/docs/operations/plan/capacity/data_sizes.html)

    `+ data size`

For example, for a record with two bins containing an integer and a string of length 20 characters, we find: 

    2 + (2 × 12) + (0 + 0) + (5 + 20) = 51 bytes.
This memory is actually split into different allocations — the record overhead plus (all) general bin overhead are in one allocation, and the type-dependent bin overhead plus data are in separate allocations per bin. Note that integer data does not need the per-bin allocation. The system heap will round allocation sizes, so there may be a few more bytes used than the above calculation implies.

#### For Data in Single-Bin Namespaces:

If a namespace configured to store data in memory is also configured as `single-bin true`, the record overhead and the general bin overhead (the first allocation) described above are not needed — this overhead is stored in the index. The only allocation needed is for the type-dependent overhead plus data. Therefore, numeric data (integer, double) has no memory storage cost — both the overhead and data are stored in the index. If it is known that all the data in a single-bin namespace is a numeric data type, the namespace can be configured to indicate this by setting `data-in-index true`. This will enable fast restart for this namespace, despite the fact that it is configured to store data in memory.


## Data Storage Required

This section should be used to determine the amount of storage, which is needed
to persist data on disk. This includes in-memory namespaces that persist to
`file` (on a filesystem) or `device` (SSD), as well as ones that store data on SSD and only keep their
indexes in memory.

### Post-4.2 Server Versions

#### Data Storage Size
In Aerospike version 4.2, a new drive format was introduced, which is more
efficient than in previous versions. The storage requirement for a single record
is the sum of the following:

- Overhead for each record:

    `35 bytes`

- If using a non-zero void-time (TTL). Note that tombstones have no expiration:

    `+ 4 bytes`

- If using a set name:

    `+ 1 byte overhead + set name length in bytes`

- If storing the record's key. Flat key size is the exact opaque bytes sent by client:

    `+ 1-3 bytes overhead + flat key size`

- Bin count overhead. No overhead for single-bin and tombstone records:

    `+1 byte for count < 127, +2 bytes for < 16K, or +3 bytes for >= 16K`

- General overhead for each bin. No overhead for single-bin and tombstone records:

    `+ 1 byte + bin name length in bytes`

- Type-dependent overhead for each bin:

    `+ 2 bytes + (1, 2, 4, 8 bytes) for integer data values 0-255, 256-64K, 64K-4B, bigger` or

    `+ 2 bytes + 8 bytes for double data` or

    `+ 5 bytes + data size for all other data types`. See [Data Size](/docs/operations/plan/capacity/data_sizes.html)

This resulting storage size should then be rounded up to a multiple of 16 bytes. For example, a tombstone record  with a set name 10 characters long and no stored key we need: 

    35 + (1 + 10) = 46 -> rounded up = 48 bytes
Or for a record in the same set, no TTL, two bins (8 character names) containing an integer and a string of 20 characters:

    35 + (1 + 10) + 1 + (2 × (1 + 8)) + (2 + 8) + (5 + 20) = 100 -> rounded up = 112 bytes

#### All Flash Index Device Space

{{#note}}
Refer to the [Primary Index Configuration](/docs/operations/configure/namespace/index/index.html#flash-index) 
page for further details.
{{/note}}

In the Aerospike All Flash configuration, every sprig requires 4KiB of index
device space, assuming that index block holds metadata for no more than 64 records.

The index device space needed if each sprig has one index block is

    4 KiB x total unique sprigs x replication factor

In the earlier example we had 4 billion unique records, a fill fraction of 1/2,
and replication factor 2. We calculated the total unique sprigs per-partition to be 32,768.

    4 KiB x 32768 x 4096 x 2 = 1 TiB index device space needed for the cluster

You can now calculate the index device space per node.

{{#note}}
You want to tolerate cluster splits and nodes going down.  Use the minimal size
of the sub-cluster you want functioning in case of a cluster split or missing nodes.
{{/note}}

    device size per node = cluster-wide index device space needed / minimal number of nodes

For the example above, assume a [`min-cluster-size`](/docs/reference/configuration/#min-cluster-size)
of 3. The index device space per node will need to be

    1 TiB / 3 = ~341GiB index device space per node

Similar to [`high-water-memory-pct`](docs/reference/configuration/#high-water-memory-pct)
for namespace RAM and [`high-water-disk-pct`](/docs/reference/configuration/#high-water-disk-pct)
for SSD data storage, there is a [`mounts-high-water-pct`](/docs/reference/configuration/#mounts-high-water-pct)
configuration param. By default it is 80% of the [`mounts-size-limit`](/docs/reference/configuration/#mounts-size-limit)
and will evict from the index device when this high water mark is breached.

In our example, we will take this high water mark into account and provision the
index device per node accordingly:

    341GiB / 0.8 = 427GiB index device space per node


### Pre-4.2 Server Versions

In Aerospike Database versions 3.1 to 4.1, storage for a single record is the sum of

- Overhead for each record:

    `64 bytes`

- If using a set name:

    `+ 9 bytes overhead + set name length in bytes`

- If storing the record's key:

    `+ 9 bytes overhead + (8 bytes (integer key) OR length of string/blob (string/blob key))`

- General overhead for each bin:

    `+ 28 bytes`

- Type-dependent overhead for each bin:

    `+ 2 bytes for integer/float data, 5 bytes for string, blob, list/map, geojson data`

- Data: size of data in all the record's bins (integer data is 8 bytes). Please see [Data Size](/docs/operations/plan/capacity/data_sizes.html)

    `+ data size`

This resulting storage size should then be rounded up to a multiple of 128 bytes. For example, to store a record with a single bin containing an integer, with a set name 10 characters long, we need:

    64 + (9 + 10) + 28 + (2 + 8) = 121 -> rounded up = 128 bytes (1 block)
Or for a record in the same set with two bins containing an integer and a string of 20 characters:

    64 + (9 + 10) + (2 × 28) + (2 + 8) + (5 + 20) = 174 -> rounded up = 256 bytes (2 blocks)

Note that for Aerospike versions prior to 3.1.0, or Aerospike versions prior to 2.7.0, storage sizes must be rounded up to a multiple of 512 bytes rather then 128 bytes. Therefore in both examples above, the resulting record storage size would be 512 bytes.

### Total Storage Required for Cluster

    (size per record as calculated above) x (Number of records) x (replication factor)

Data can be stored in RAM or on flash storage (SSD). You should not exceed 50-60% capacity on your SSDs. You can use one of [our recommended SSDs](https://www.aerospike.com/docs/operations/plan/ssd/ssd_certification.html) or test/certify your own SSD using the [Aerospike Certification Tool](http://aerospike.github.io/act/).

## Throughput (bytes)

    (number of records to be accessed in a second) × (the size of each record)
Calculate your desired throughput so that the cluster would continue to work even if one node goes down (i.e., we want to make sure that each node can handle the full traffic load).

## Provisioning

See [Provisioning a Cluster](/docs/operations/plan/capacity/provisioning.html) for examples.

