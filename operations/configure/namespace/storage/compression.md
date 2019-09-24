---
title: Configuring Storage Compression
description: Configure Storage Compression
---

## Storage Compression

Aerospike's storage compression feature provides lossless compression
of records written to persistent storage. Aerospike supports three
different compression algorithms, which provide superior compression
and decompression speed:

  * _LZ4_ - [http://lz4.org](http://lz4.org). Aerospike supports the
    widely deployed, speed-optimized version of _LZ4_. There is also
    an _HC_ ("high compression") variant, which is slower and not
    supported.

  * _Snappy_ -
    [https://google.github.io/snappy/](https://google.github.io/snappy/).

  * _Zstandard_ -
    [https://facebook.github.io/zstd/](https://facebook.github.io/zstd/).

Compression is only applied to records that are written to persistent
storage, e.g., SSDs. Record data in memory (RAM) is not compressed.

The trade-off of compression is increased transaction latency and higher CPU 
load, because in addition to accessing the SSD, compression and decompression
have to take place. Typically, write latency is affected more than read latency,
because compression is more CPU-intensive than decompression.

An algorithm's performance - speed as well as compression ratio - is
highly specific to the data to be compressed. When picking an
algorithm, it is good practice to evaluate each algorithm's
performance with records that are representative of real-world data.

An empirical approach towards evaluating the optimal compression algorithm is to configure different compression algorithms on multiple servers and then reviewing 
the compression percentages between the servers.

Note that compression does not allow for records larger than the
configured write block size; it's the uncompressed record size that
counts for the record size check. This ensures that a record will
always fit into a write block, even when compression is disabled
again later.

Although different (or no) compression settings may be made for the same namespace on different nodes, this 
should only be considered as a temporary approach during a transition towards a cluster where that namespace has the same compression settings on each node.

## Configuration

Storage compression is configured per namespace per node, using two
configuration directives,
[`compression`](/docs/reference/configuration/#compression) and
[`compression-level`](/docs/reference/configuration/#compression-level).

  * [`compression`](/docs/reference/configuration/#compression) selects the compression algorithm. Valid parameters
    are `none`, `lz4`, `snappy`, and `zstd`.

    If this setting is not specified, it defaults to `none`, i.e., no
    compression.

  * [`compression-level`](/docs/reference/configuration/#compression-level)
    controls the trade-off between compression
    speed and compression ratio.

    The valid parameter values `1` through `9` are used as a scale.
    On one end of the scale, `1` selects a faster and less 
    efficient compression. On the other end, `9` selects a slower and 
    more efficient compression.

    For compression algorithm `zstd`, in Aerospike Server versions prior to 4.6.x, 
    if this setting has never been specified when using `compression zstd`,
    a default flag of `0` is displayed and the `compression-level` of `9` will be used.
    
    For compression algorithm `zstd`, in Aerospike Server versions 4.6.x or newer, 
    if this setting has never been specified when using `compression zstd`,
    a default flag of `9` is displayed and the `compression-level` of `9` will be used.

    This setting is only honored by _Zstandard_. It does not have any
    effect for _LZ4_ and _Snappy_.

The configuration directives belong to a namespace's `storage-engine`
section, for example:

```
namespace test {
    ...
    storage-engine device {
        device /dev/sda1
        ...
        compression zstd
        compression-level 1
    }
    ...
}
```

Both settings can also be configured dynamically, for example:

```
asinfo -v 'set-config:context=namespace;id=test;compression=zstd'
asinfo -v 'set-config:context=namespace;id=test;compression-level=1'
```

The currently configured compression settings are consulted whenever a
newly inserted or an updated record is compressed on its way to
storage. The compression settings then control how this record is
compressed.

This means that changing the compression settings doesn't affect
existing records on storage. It only affects records that are newly
inserted or updated after the compression settings are changed. If you
switch [`compression`](/docs/reference/configuration/#compression)
 from `lz4` to `snappy`, then existing records
will remain compressed with _LZ4_, until they get updated. Then they
will be compressed according to the current settings, i.e., with
_Snappy_.

This also means that storage may contain a mix of uncompressed records
as well as records compressed with any of the three compression
algorithms. This is fine, because decompression works independently
from the [`compression`](/docs/reference/configuration/#compression)
 setting; it always applies the correct
algorithm when decompressing a record, i.e., the algorithm that was
used to compress it.

Note that a record may not be compressible, so that applying
compression would increase its size. In this case, the record is
stored uncompressed instead of storing the larger compressed version.

In order to change the compression settings for existing records,
perform the following steps on each cluster node in a rolling
fashion. Note that this assumes that you run at least with replication
factor 2 - otherwise data loss will occur.

  * Stop the Aerospike server on the cluster node to be reconfigured.

  * Change the compression settings in `aerospike.conf`.

  * Clear the data storage of the cluster node, e.g., by using `dd` on
    the storage drives. This will remove all records compressed with
    the previous settings.

  * Restart the Aerospike server.

  * Wait for data migration to finish. Data migration will re-fill the
    node and the incoming records will be compressed with the changed
    compression settings.

Repeat this process for all nodes in the cluster.

Alternatively, your application can trigger a re-write of each record
in the cluster. These record re-writes can also be triggered by touch
operations. This would entail the following steps:

  * Dynamically change the compression settings on all cluster nodes.

  * Read each record in the cluster and write it back. Alternatively,
    touch each record.

## Statistics

Aerospike reports the current compression ratio as part of the
namespace statistics.

```
$ asinfo -l -v 'namespace/test' | grep device_compression_ratio
device_compression_ratio=1.000
```

The given number is the average _compressed size : uncompressed size_
ratio. Thus, the above value, _1.000_, means no compression at all. In
contrast, _0.100_ would indicate a ratio of _1 : 10_, i.e., a size
reduction by 90%.

Note that `device_compression_ratio` will be included in the namespace
statistics unless [`compression`](/docs/reference/configuration/#compression) is set to `none`.

The compression ratio is a moving average. It is calculated based on
the most recently written (= most recently compressed) records. Read
records do not factor into the ratio.

If the written data - and how compressible it is - changes over time,
then the compression ratio will change with it. In case of a sudden
change in data, the indicated compression ratio may lag behind a
bit. As a rule of thumb, assume that the compression ratio covers the
most recently written 100,000 to 1,000,000 records.

In particular, this means that the compression ratio might not
accurately reflect the compression ratio across all records on
storage. The actual storage savings across all records might be higher
or lower than the current compression ratio, which just covers the
most recently written records.

When evaluating different compression settings for real-world data,
proceed as follows:

  * Set [`compression`](/docs/reference/configuration/#compression) and 
[`compression-level`](/docs/reference/configuration/#compression-level).

  * Write 1,000,000 representative records to make the ratio converge
    to reflect the actual compression ratio of these records.

  * Query the compression ratio via the above info command.

Repeat these steps for all compression settings to be evaluated.
