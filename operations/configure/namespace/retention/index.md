---
title: Namespace Retention Configuration
description: Learn how to configure Aerospike properly in order to manage when to remove records based on the TTL specified by the application.
---

Aerospike may be configured to **expire** or **evict** least recently updated
data. The *expiration* and *eviction* algorithms use the record's **TTL**
(Time To Live) value to determine eligibility for removal. The frequency of
*expiration* and *eviction* passes are determined by the `nsup-period`
configuration parameter. An *eviction* run happens when your data exceeds either
[`high-water-disk-pct`](/docs/reference/configuration/#high-water-disk-pct) or
[`high-water-memory-pct`](/docs/reference/configuration/#high-water-memory-pct)
(or, if applicable,
[`mounts-high-water-pct`](/docs/reference/configuration/#mounts-high-water-pct))
in order to allow the cluster to continue processing new writes. *Eviction* may
be thought of as "early *expiration*" since *eviction* only removes data that
has a *TTL* set and chooses the soonest to *expire* records to *evict* first
(in namespaces with similar ttl across records).

{{#note}}
When evictions are in effect, Aerospike groups records by buckets based on their expiration time and evicts randomly within each bucket, emptying a bucket before moving to the next one. In Aerospike Server 3.8.1 and up, a more granular eviction histogram has been introduced along with a configuration parameter: [`evict-hist-buckets`](/docs/reference/configuration/#evict-hist-buckets). In Aerospike Server 3.7.5.1 or earlier there are always 100 buckets. The bucket boundaries are determined based on the record with the longest expiration time in the namespace. The longer the expiration time of this record, the less precise the eviction will be since the buckets will have broader ranges.
{{/note}}

{{#note}}
Prior to Aerospike Server 4.5.1, expirations and evictions are accomplished by internally generating delete transactions, including replica deletion via the fabric. For heavy expiration and eviction loads, these transactions can noticeably impact overall performance.
<br>
As of Aerospike Server 4.5.1, expirations and evictions are accomplished by deleting records locally on each node, without generating any transaction load.  This enables use cases with heavy expiration or eviction loads which were not possible before.
<br>
This new methodology has a somewhat stronger dependence on clocks being synchronized across nodes in a cluster.  As of Aerospike Server 4.5.1, for each namespace where nsup is enabled (i.e. [`nsup-period`](/docs/reference/configuration/#nsup-period) not zero) writes will be suspended if cluster clock skew exceeds 40 seconds.
<br>
Also, as of Aerospike Server 4.5.1, if eviction thresholds are not configured the same on all nodes in the cluster, the node with the lowest threshold will drive eviction across the cluster. When a node breaches its eviction threshold, it broadcasts a cutoff void-time to the rest of the cluster. [Prior to 4.5.1, a node with a lower threshold would evict heavily from the partitions for which it is master.]
{{/note}}

{{#info}}
See 
[FAQ What are Expiration, Eviction and Stop-Writes?](https://discuss.aerospike.com/t/faq-what-are-expiration-eviction-and-stop-writes/2311/1)
{{/info}}

### Namespace Data Retention
If configured, Aerospike will manage when to remove records based on the TTL
specified by the application. Aerospike never evicts or expires data that has
been stored with a TTL of 0, so depending on your use case it may be desirable
to set records to not expire and manage the data from the application.

The following example shows the applicable namespace data retention parameters
and a short comment describing how they are used.
```bash
namespace <namespace-name> {
    default-ttl <VALUE>             # How long (in seconds) to keep data after
                                    # it is written

    high-water-disk-pct <PERCENT>   # How full may the disk become before the
                                    # server begins eviction

    high-water-memory-pct <PERCENT> # How full may the memory become before the
                                    # server begins eviction

    stop-writes-pct <PERCENT>       # How full may the memory become before
                                    # we disallow new writes
    
    index-type flash {                  # If applicable (as of Aerospike 4.3.0)
        mounts-high-water-pct <PERCENT> # How full flash index storage may become
                                        # before server begins eviction
        ...
    }

    index-type pmem {                   # If applicable (as of Aerospike 4.5.0)
        mounts-high-water-pct <PERCENT> # How full pmem index memory may become
                                        # before server begins eviction
        ...
    }

    ...
}
```

The following example shows additional namespace data retention parameters to tune
data expiration and eviction, with comments describing usage.
```bash
namespace <namespace-name> {
    nsup-period <SECONDS>           # As of Aerospike 4.5.1
                                    # Maximum time between starting successive
                                    # rounds of expiration or eviction - a value
                                    # of zero disables expiration and eviction

    nsup-threads <NUMBER>           # As of Aerospike 4.5.1
                                    # How many threads per round of expiration
                                    # or eviction

    evict-tenths-pct <NUMBER>       # Fraction of evictable records to delete
                                    # per round of eviction (e.g. 5 means
                                    # delete 0.5 percent of evictable records)

    ...
}
```

### Per-Set Data Retention

#### Set Disable Eviction

As of Aerospike 3.6.1, a set can be protected from evictions. The following example shows how to protect a set from evictions:

```bash
namespace <namespace-name> {
    ...
    set <set-name> {
        set-disable-eviction true     # Protect this set from evictions.

    }
}
```

This can be set in the configuration as well as [dynamically](/docs/reference/configuration/index.html#set-disable-eviction).

#### Set Stop Writes Count

As of Aerospike 3.7.0.1, a set can have a set-stop-writes-count to limit the number of records that can be written to it. This can be set in the configuration as well as [dynamically](/docs/reference/configuration/index.html#set-stop-writes-count).

```bash
namespace <namespace-name> {
    ...
    set <set-name> {
        set-stop-writes-count 5000     # Limit number of records that can be written to this set to 5000.

    }
}
```


### Where to Next?
- Configure [Storage Engine](/docs/operations/configure/namespace/storage) which determines if and where records are
  persisted to.
- Configure [Data Durability Policy](/docs/operations/configure/namespace/durability) which determines how many
  replica copies of a record to keep in the cluster.
- Or return to [Configure Page](/docs/operations/configure).
