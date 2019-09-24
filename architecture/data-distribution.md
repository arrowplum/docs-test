---
title: Data Distribution
description: Aerospike synchronously replicates data to provide high performance and high availability with database cluster partitioning.
assets: /docs/architecture/assets
---

Aerospike Database uses a **Shared-Nothing** architecture, where:
- Every node in the Aerospike cluster is identical.
- All nodes are peers.
- There is no single point of failure.

Using the Aerospike **Smart Partitions™** algorithm, data distributes evenly across all nodes in the cluster.

### Partitions

In the Aerospike Database, a namespace is a collection of data that has common storage, such as on a specific drive, and policies, such as the number of replica copies for each record in the namespace. Each namespace is divided into 4096 logical partitions, which are evenly distributed between the cluster nodes. This means that if there are _n_ nodes in the cluster, each node stores ~1/_n_ of the data.

#### Record Distribution to Partitions

Aerospike uses a deterministic hash process to consistently map a record to a single partition.

To determine record assingment to a partition, the record's key (of any size) is hashed into a 20-byte fixed-length digest using RIPEMD160. Using 12 bits of this digest, the partition ID of the record is determined.

{{#note}}
RIPEMD160, which is a field-tested, extremely random hash function, ensures that records distribute very evenly on a partition by partition basis. The partitions follow a normal distribution across the different nodes a cluster (within 3 standard deviations from the mean).
{{/note}}

#### Partition Distribution to Cluster Nodes

Partition distribution in Aerospike has the following characteristics

 * Aerospike uses a random hashing method to ensure that partitions distribute evenly to the cluster nodes. There is no need for manual sharding.
 * All of the nodes in the cluster are peers – there is no single database master node that can fail and take the whole database down.
 * When nodes are added or removed, a new cluster will form and its nodes will coordinate to evenly divide partitions between themselves. The cluster will then automatically re-balance.

Because data distributes evenly (and randomly) across cluster nodes, there are no hot spots or bottlenecks where one node handles significantly more requests than any other node. For example, in the United States, many last names begin with R. If data is stored alphabetically, the server handling the last names beginning with R has a lot more traffic than the server handling last names beginning with X, Y, or Z. Random data assignment ensures a balanced server load.

### Data Replication and Synchronization

For reliability, Aerospike replicates partitions on one or more nodes. One node becomes the data master for reads and writes for a partition, while other nodes store its replica partitions.

<div style="float: right" >
![]({{book.assets}}/ARCH_shared_nothing_small.png)
</div>

This illustrates a 4-node Aerospike cluster, where each node is the data master for roughly 1/4 of the data _AND_ each node is the replica for 1/4 of the data. One node is the data master. Data distributes across all other nodes as replicas. For this example, if node 1 becomes unavailable, replicas from node #1 are spread across the other nodes.
<br />
<br />
<br />
{{#note}}
The replication factor is configurable; however, it cannot exceed the number of nodes in the cluster. More replicas equals better reliability, but creates higher cluster demand as write requests must go to all replicas. Most deployments use replication factor of 2 (one master copy and one replica).
{{/note}}

Synchronous replication provides a higher level of correctness in the face of no network faults. A write transaction propagates to all replicas before committing the data and returning results to the client. In rare cases during cluster reconfiguration when the Aerospike Smart Client may have sent the request to the wrong node because it is briefly out of date, the Aerospike **Smart Cluster™** transparently proxys the request to the right node. When a cluster is recovering from partitioning, there may be writes which have been applied in conflict to different partitions. In this case, Aerospike applies a heuristic to choose the most likely version, which is it resolves any conflicts that occurred between different copies of the data. By default, the version with the largest number of changes ( highest generation count ) is chosen, although the version with the most recently modified time can be chosen. The correct choice will be determined by the data model.

#### Aerospike Cluster with No Replication

In the Aerospike Database, having NO replicated data is replication factor = 1&mdash;there is only a single copy of the database. 

<div style="float: right" >
{{#figure "" "Replication Factor = 1; Two nodes in a four-node cluster no replication" size="large"}}
![]({{book.assets}}/ARCH_shared_nothing2_small.png)
{{/figure}}
</div>

This illustrates two nodes of a four-node cluster that has a total 4096 partitions. Each node contains a random assignment of 1/4th of the data (1024 partitions). Each server/node manages this collection of partitions.
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
Each node is the data master for 1/4th of the data partitions (nodes are data masters when it is the primary source for reads and writes to that data). The Aerospike Smart Client is location-aware. It knows where each partition is located so that the data retrieval is achieved in a single hop. Every read and write request is sent to the data master for processing. The Smart Client reads records and sends a request to the correct data master node for that record.

#### Aerospike Cluster with Replication

In the Aerospike Database, a replication factor = 2 means storing two copies of the data: _master_ and _replica_.

<div style="float: right" >
{{#figure "" "Replication Factor = 2; Two nodes in a four-node cluster using replication" size="large"}}
![]({{book.assets}}/ARCH_shared_nothing_small.png)
{{/figure}}
</div>

This illustrates that each node is the data master for 1/4 of the data (1024 partitions) _AND_ that each node is the replica for 1/4 of the data (1024 partitions). Note that the data for one data master is distributed across all other cluster nodes as replicas. If node #1 becomes unavailable, the replicas of the data from node #1 distribute to all other cluster nodes.
<br />
<br />
<br />
<br />
<br />
Again, the Smart Client reads records and sends a request to the correct data master node for that record. Write requests are also sent to the correct node. When a node receives a write request, it saves the data and forwards the write request to the replica node. Once the replica node confirms a successful write and the node writes the data itself, a confirmation returns to the client.

### Automatic Rebalancing

The Aerospike data rebalancing mechanism ensures that query volume distributes evenly across all cluster nodes, and is persistent during node failure. The system is continuously available. Rebalancing does not impact cluster behavior. The transaction algorithms integrated with the data distribution system ensure that there is one consensus vote to coordinate a cluster change. Voting per cluster change, instead of per transaction, provides higher performance while maintaining shared-nothing simplicity.

Aerospike allows configuration options to specify how fast rebalance proceeds. Temporarily slowing transactions heals the cluster more quickly. If you need to maintain transactional speed and volume, the cluster rebalances more slowly.

{{#todo}}
Add figure for migrations
{{/todo}}

During rebalance, Aerospike does not retain full replication factors of all partitions. Some in-transit partitions temporarily become single replica, to provide maximal memory and storage availability as the cluster rebalances to new stability.

By not requiring operator intervention, the cluster self-heals even at the most demanding times. For example, in one customer deployment a rack circuit breaker tripped, and one node of an 8-node cluster went down. No operator intervention was required. After several hours the fault was corrected and the rack came back online. Operators never had to take special steps to maintain the Aerospike cluster.

In Aerospike, capacity planning and system monitoring manage virtually any failure with no loss of service. You can configure and provision your hardware capacity, and set up the replication/synchronization policies so that the database recovers from failures without affecting users.

### Traffic Saturation Management

The Aerospike Database monitoring tools let you evaluate bottlenecks. Network bottlenecks decrease database throughput capacity, making requests slow.

### Capacity Overflows

On storage overflow, the Aerospike stop-write limit prevents new record writes. Replica and migration writes, as well as reads, continue processing. So, even beyond optimal capacity, the database does not stop handling requests. It continues to do as much as possible to continue processing user requests.

{{#todo}}
#### sync replication
#### async replication
### What happens when write post failure 
#### Duplicate resolution
{{/todo}}
