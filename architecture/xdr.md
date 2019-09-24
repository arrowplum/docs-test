---
title: XDR Architecture
assets: /docs/architecture/assets
---

<div style="float: left" >
{{#figure "" "Data replicates to geographically diverse clusters" size="large"}}
![]({{book.assets}}/XDR_image_small.png)
{{/figure}}
</div>

The Aerospike Cross Datacenter Replication (XDR) feature is designed to synchronize clusters over higher-latency (typically: WAN) links. XDR is asynchronous, but the replication lag is usually below one second.

Each client write is logged on the written-to cluster (the _local_ cluster), and this log is used to replicate modified data from the local cluster to one or more _remote_ clusters. Replication can be selective: Different namespaces or sets can be replicated to different remote clusters.

XDR offers the same redundancy as a single Aerospike cluster. The master for any partition is responsible for replicating writes for that partition. When the master fails and a replica is promoted to master, the new master takes over where the failed master left off.

In addition to cluster node failures, XDR gracefully handles failures of a network link or of a remote cluster. First, replication via the failed network link or to the failed remote cluster is suspended. Then, once the issue has been resolved and things are back to normal, XDR resumes for the previously unavailable remote cluster and catches up. Replication via other (intact) links to other (intact) remote clusters remains unaffected at all times.

When XDR logs a client write, it only logs the key digest of the written record; it does not log the record content, i.e., its bins. Accordingly, the log file is referred to as the _digest log_. To replicate client writes, XDR reads key digests from the digest log, retrieves the corresponding records from storage, and writes the retrieved records to the remote cluster(s). In case of hot records, i.e., records that change very frequently, optional deduplication coalesces multiple local writes to the same record into a single remote write.

<div style="float: right" >
{{#figure "" "Aerospike XDR module" size="large"}}
![]({{book.assets}}/xdr_diagram.jpg)
{{/figure}}
</div>

XDR comprises the following components:
- _Logger_, which logs local writes to the digest log on the local cluster.
- _Shipper_, which replicates writes for partitions for which a node is the master.
- _Node Failure Handler_, which takes over another master's replication obligation, should that master fail.
- _Link Failure Handler_, which makes a remote cluster catch up once XDR for that cluster resumes after a network link or remote cluster failure.

<br/>
<br/>
<br/>

#### Logger

The logger stores only minimal information about each client write&mdash;the key digest of the written-to record&mdash;in the digest log file. This keeps the size of the digest log file manageable.

The digest log is a ring buffer file. The most-recent data overwrites the oldest data (if the buffer is full). This caps the file size, but also introduces a capacity planning issue. Be sure to configure a digest log file that is large enough to avoid losing any data waiting to ship. It is important to plan for long term outages due to problems communicating between your datacenters. Configure your digest log file size on each cluster node, such that it is able to retain the key digests of all writes to that node (master as well as replica) even in case of multi-day outages.

{{#note}}
You cannot change the size of the digest log file without deleting the contents of the old file. If you change the digest log file size configuration, you must also delete the existing digest log file before starting XDR.
{{/note}}


#### Shipper

The Shipper reads the digest log and ships the modified records. To the remote cluster, the Shipper looks like any other client application; the data ships to the remote cluster in a normal write operation.

The XDR ship throughput to a remote site is predominantly dependent on the client write throughput on the local cluster, and the available bandwidth between the local and remote datacenters.

{{#note}}
If the client write throughput is consistently above the XDR ship throughput, XDR cannot synchronize all data on the remote cluster. Modified records awaiting shipment to the remote cluster back up and may be dropped.
{{/note}}


Typically, production environments have load cycles with peak times. Ensure that you have sufficient available bandwidth for pending records to ship during low activity periods.

#### Failure Handlers

The Failure Handlers ensure that the system ships data in case of the following failures:

- Local cluster node failure - Node Failure Handler
- Communications failure (for example, link to remote down) - Link Failure Handler

XDR ensures that data is not lost during single-node failures. Data can be lost only in extreme situations such as digest log overflow or if a master and replica fail simultaneously. On failure of a master node, promoted replica nodes start to ship their copies of the failed master's data.

#### Workflow

1. At startup, the local cluster discovers namespaces that must be synchronized with the remote cluster.
 - Note that the number of remote cluster nodes can differ from the source cluster.
1. Client reads are served by local nodes (using local replication). XDR doesn't change anything about that.
1. Each client write is stored in the digest log on the master node as well as the digest logs on the replica nodes for the write. In this way, the digest log is subject to the same degree of redundancy on the local cluster as is the written data. When a master fails, replicas can take over.
 - Writes are confirmed when the master and replica have successfully written the data.
1. XDR asynchronously ships the client writes in the digest log to the remote datacenter to synchronize the remote cluster with the local cluster.
 - XDR reads a key digest from the digest log.
 - XDR reads the corresponding record from storage.
 - XDR writes the record to the remote cluster.
 - In case the record cannot be written to the remote cluster, its key digest is re-logged to the digest log. In this way, the record will be processed again at a future point in time.

{{#note}}
There is no one-to-one correspondence between nodes of the local cluster and nodes of the remote cluster. Even if both have the same number of nodes, partitions may be distributed differently across nodes in the remote cluster than they are in the local cluster. Hence, every master node of the local cluster may write to any remote cluster node. Just like any other client, XDR writes a record to the remote master node for that record.
{{/note}}


### XDR Topologies

<div style="float: right" >
{{#figure "" "Simple active-passive topology" size="large"}}
![]({{book.assets}}/XDR_active_passive_small.png)
{{/figure}}
</div>

XDR configurable topologies are:

- simple active-passive
- simple active-active
- star replication
- combination topology

<br />
<br />
<br />
Star topology allows one datacenter to simultaneously replicate data to multiple datacenters.

#### Simple Active-Passive

In active-passive topology, client writes happen only to one cluster, let's call it cluster A. The other cluster, cluster B, is a stand-by cluster that can be used for reads. Client writes are shipped from cluster A to cluster B. However, client writes to cluster B are not shipped to cluster A. Additionally, XDR offers a way to completely disable client writes to cluster B, instead of just not shipping them.

A typical scenario would be to offload performance-intensive analysis of data from a main cluster (cluster A) to an analysis cluster (cluster B).

#### Simple Active-Active

<div style="float: right" >
{{#figure "" "Simple active-active topology" size="large"}}
![]({{book.assets}}/XDR_active_active_small.png)
{{/figure}}
</div>

In active-active topology, client writes can happen to both clusters. When writes happen on one cluster, they are forwarded to the other. A typical use case for a simple active-active topology is when client writes to a record are strongly associated with one of the two clusters and the other cluster acts as a hot backup.
<br />
<br />
<br />
<br />
XDR ensures that shipped records are not forwarded beyond the remote cluster, which avoids an infinite loop of writes between clusters. Since XDR allows client writes to happen to both clusters, two clients may simultaneously write to the same key on both clusters, which leads to inconsistent data. As the Aerospike database is agnostic of the application data, it cannot auto-correct this inconsistency. Subsequent writes to the key on just one of the two clusters will cause this write to be shipped to the other cluster and thus resolve this issue. But still, if the same record may be simultaneously written to on two different clusters, this topology is not suitable.

Another good use case for the active-active topology is a company with users spread across North America. Traffic is divided between the West and East Coast datacenters. While a West Coast user can travel to the East Coast and be serviced by the East Coast datacenter, it is unlikely that writes for this user will occur simultaneously in both datacenters.

#### Star Replication

<div style="float: right" >
{{#figure "" "Star replication" size="large"}}
![]({{book.assets}}/XDR_star_small.png)
{{/figure}}
</div>

To enable XDR to ship data to multiple destination clusters, simply specify multiple destination clusters in the namespace configuration. Specify each destination cluster on a separate line using the `xdr-remote-datacenter` configuration parameter. Star replication topology is most commonly used when data is centrally published and replicated in multiple locations for low-latency read access from local systems.
<br />
<br />

#### Combination Topology

<div style="float: right" >
{{#figure "" "Combination topology" size="large"}}
![]({{book.assets}}/XDR_complex_small.png)
{{/figure}}
</div>

This illustrates a combination of the above topologies.
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

### Failure Handling

XDR manages the following failures:

- Local node failure
- Remote link failure
- Combinations of the above

#### Local Node Failure

In case of a local node failure, the Node Failure Handlers on the replica nodes take over the shipping responsibilities of the Shipper on the failed master node. The replicas have a copy of the master's records as well as a copy of the relevant portions of the master's digest log file, which allows them to pick up where the failed master left off.

#### Communications Failure

If the connection between the local and the remote cluster drops, the Link Failure Handler on each master node records the point in time when the link went down and shipping is suspended for the affected remote datacenter. When the link becomes available again, shipping resumes in two ways:

- New client writes are shipped by the Shippers, just as they were before the link failure.
- Client writes that happened during the link failure, i.e., client writes that were held back while shipping was suspended for the affected remote cluster, are shipped by the Link Failure Handlers.

In short, the Link Failure Handlers are responsible to make the affected remote cluster catch up and receive all the client writes that happened during the failure.

When XDR is configured with star topology, a cluster can simultaneously ship to multiple datacenters. If one or more datacenter link drops, XDR continues to ship to the remaining available datacenters.

XDR can also handle more complex scenarios, such as local node failures combined with remote link failures.

#### Combination Failure

XDR also seamlessly manages combination failures such as local node down with a remote link failure, link failure when XDR is shipping historical data, and so on.

### Compression

XDR can compress shipment data to save bandwidth between datacenters. You can configure [`xdr-compression-threshold`](/docs/reference/configuration/index.html#xdr-compression-threshold), so that only records larger than this minimum size are compressed.

### Per-namespace Shipping

<div style="float: right" >
{{#figure "" "Flexible clustering" size="large"}}
![]({{book.assets}}/XDR_ns_shipping_small.png)
{{/figure}}
</div>

Aerospike nodes can have multiple namespaces. You can configure different namespaces to ship to different remote clusters. In this illustration, _DC1_ is shipping namespaces _NS1_ and _NS2_ to _DC2_, and shipping namespace _NS3_ to _DC3_. Use this flexibility to set different replication rules for different data sets. In this example, writes to namespace _NS1_ could loop around the clusters forming a ring. To avoid this, use the XDR option that stops forwarding writes coming from another XDR.
<br />
<br />
<br />
<br />
<br />

### Sets

For fine-grained control over which data to ship, configure XDR to ship certain sets (RDBMS _tables_) to a datacenter. The combination of namespace and set determines whether to ship a record. Use sets if not all data in a namespace in a local cluster needs to be replicated in other clusters.

### Heterogeneous Cluster Synchronization

XDR works for clusters of different size, operating system, storage media, and so on. The XDR failure handling capability allows the source cluster to change size dynamically. It also works when multiple-destination datacenters go up and down often.

### Remote Cluster in a Local Datacenter

While the most common deployment has local and remote clusters in different datacenters, sometimes the remote cluster may be in the same datacenter. Common reasons for this are:

- The remote cluster is only for data analysis.
 Configure the remote cluster for passive mode and run all analysis jobs in that cluster. This isolates the local cluster from the workload and ensures availability.
- Multiple availability zone datacenters (such as Amazon EC2) to ensure that if there is a large-scale problem with one availability zone, the other is up.
 Administrators that have had problems with availability zone outages may elect to have clusters in multiple availability zones within a datacenter. For best performance, all nodes in a cluster must belong to the same availability zone.

### Shipping Deletes

XDR ships deletes as well as writes. Delete shipping is important to sync objects with all datacenters. Client-initiated deletes ship by default. You can configure whether or not to ship deletes generated via eviction of objects.
