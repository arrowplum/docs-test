---
title: Strong Consistency mode
description: Use per-transaction consistency levels.
---

As per Eric Brewer's CAP theorem, Aerospike is, as of version 3.0, an AP
database - that is, one that provides availability instead of consistency in
various network partition situations. Aerospike 3.0 does not provide several
features that are required in order to maintain consistent replication of
records during a transaction, and instead, allows data to be available, and
writes to be taken, in cases that will eventually create data conflicts.
The process of choosing one version of a record over another will likely lose
written data.

As of Aerospike 4.0, Aerospike supports both AP (Available and Partition
Tolerant) as well as CP (Consistent and Partition Tolerant) mode. These modes
can be configured on a namespace-by-namespace basis through the 
[strong-consistency](/docs/reference/configuration/index.html?show-removed=1#strong-consistency) 
configuration parameter.

## Available mode
As a distributed database, Aerospike supports automatic data replication. Often,
a database maintains multiple identical copies of every record, known as a
replication factor. Aerospike is often run with two copies of data,
i.e. _replication factor 2_. The Aerospike server supports other replication
factors, which are configurable on a per-namespace basis. See
[Data Distribution](/docs/architecture/data-distribution.html).

Automatic data replication gives the system the advantage of high performance
and availability, but at the cost of increased transaction latency. During good
operational network circumstances, the increased latency cost is quite low, but
can be an issue during cluster reconfiguration or during network faults, when
all replicas may not yet be synchronized to the latest copy. Write transaction
latency is affected by replication factor because of the additional
intra-cluster communication required to commit changes to the replica(s)
residing on non-master server node(s.)

By default, Aerospike applies writes immediately to replicas. When the network
is operating correctly, this will result in data consistency by involving all
replicas of a record during each transaction, and not creating situations where
there are stale or dirty reads. In the case where network partitions are
occurring, Aerospike will prioritize availability over consistency --- Aerospike
will allow reads and writes in every sub-cluster.

The default Aerospike server-client behavior provides these behaviors:

- Read transactions only consult a single replica (usually the master) even
  during cluster reconfiguration.
- Write transactions (including deletes and UDF applications) will write
  locally, then write all replicas synchronously before successfully returning
  from the transaction.

These options provide a trade-off of higher correctness at a slight cost of
network overhead for an availability-focused database. Latencies will remain
low, and higher levels of correctness will be observed compared to databases
which do not replicate synchronously. However, this does not provide
consistency.

For reads using non-default settings, more stale reads will be seen, because a
later read may return an earlier value for the record (for example, while the
cluster is undergoing reconfiguration and the master replica does not yet have
the latest written copy).

For writes using non-default-settings, more writes may be lost because a 
transaction may return success when data is written to the master but not 
successfully replicated.

For maximum application flexibility, Aerospike provides
per-transaction-selectable data policies. Use these replica-related policies to
allow applications to be tuned for desired performance versus data consistency
levels.

## Strong Consistency mode
The strong consistency guarantee states that all writes to a single record will
be applied in a specific order (sequentially), and writes will not be
re-ordered or skipped. 

In particular, writes that are acknowledged as committed have been applied, and
exist in the transaction timeline in contrast to other writes to the same
record. This guarantee applies even in the face of network failures, outages,
and partitions. Writes which are designated as "timeouts" (or "InDoubt" from the
client API) may or may not be applied, but if they have been applied are only
observed as such.

Aerospike's strong consistency guarantee is per-record, and involves no
multi-record transaction semantics. Each record's write or update will be atomic
and isolated, and ordering is guaranteed using a hybrid clock.

Aerospike provides both full Linearizable mode, which provides a single linear
view among all clients that can observe data, as well as a more practical
Session Consistency mode, which guarantees an individual process sees the
sequential set of updates. These two read policies can be chosen on a
read-by-read basis, thus allowing the few transactions that require a higher
guarantee to pay the extra synchronization price, and are detailed below. 

In the case of a "timeout" return value - which could be generated due to
network congestion, external to any Aerospike issue - the write is guaranteed to
be written, or not written.

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_21.png)
{{/figure}}
 
Figure 21: Split Brain Cluster

Most systems for providing such strong consistency require a minimum of three
copies to ensure that consistency properly, based on Lamport's proof that a
consistency algorithm requires three copies of data. So, if a cluster splits as
shown in Figure 21, one of the two sub parts can allow writes if it has a
majority (two out of three) copies of the data item. 

Aerospike optimizes this further by regularly allowing storing only two copies
but using an adaptive scheme that adds more write copies on the fly in
situations where they are necessary, thus optimizing the performance in the
normal case while incurring a small amount of overhead in edge cases that rarely
occur. First in strong consistency mode, Aerospike defines a roster for the
cluster.

### Roster
This defines the list of nodes that are part of the cluster in steady state.
When all the roster nodes are present and all the partitions are current, the
cluster is in its steady state and provides optimal performance. As we described
in the partition algorithm earlier, the master and replica partitions are
assigned to nodes in a cluster using a random assignment of partitions to nodes.
In the case of strong consistency, these partitions are referred to as
roster-master and roster-replica. For the purpose of simplifying the discussion,
we will restrict ourselves to a system with replication factor set to 2. Every
partition in the system will have one master and one replica. 

First some terminology:

`roster-replica` – For a specific partition, the `roster-replica` refers to the
node that would house the replica of this partition if all nodes in the roster
were part of the single cluster, i.e., the cluster was whole

`roster-master` – For a specific partition, the `roster-master` refers to the
node that would house the master of this partition if all nodes in the roster
were part of the single cluster, i.e., the cluster was whole

The following rules are now applied to the visibility of partitions:

1. If a sub cluster (a.k.a. split-brain) has both the roster-master and
   roster-replica for a partition, then the partition is active for both reads
   and writes in that sub cluster
1. If a sub cluster has a majority of nodes and has either the roster-master or
   roster-replica for the partition within its component nodes, the partition is
   active for both reads and writes in that sub cluster
1. If a sub cluster has exactly half of the nodes in the full cluster (roster)
   and it has the roster-master within its component nodes, the partition is
   active for both reads and writes

The above rules also imply the following:

100% availability on rolling upgrade: If a sub cluster has fewer than
replication factor number of nodes missing, then it is termed a super-majority
sub-cluster and all partitions are active for reads/writes within the cluster.

100% availability on two-way split-brain: If the system splits into exactly two
sub clusters, then all partitions are active for reads and writes in one or the
other sub cluster (we will later show how to use this in a creative way for a 
rack-aware based HA architecture).

Consider as an example, partition p in a 5-node cluster where node 4 is the
roster-replica for p and node 5 is the roster master for p. You can see below in
Figure 22, Figure 23, Figure 24 and Figure 25 examples of when a partition is
available or not in various partitioning situations.

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_22.png)
{{/figure}}
 
Figure 22: All roster replicas are active, whole cluster

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_23.png)
{{/figure}}
 
Figure 23: Minority sub-cluster has both roster-master and roster-replica, p is
active

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_24.png)
{{/figure}}

Figure 24: Roster-replica is in majority sub-cluster, becomes master, p is
active, new replica in node 3

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_25.png)
{{/figure}}

Figure 25: Roster-master and roster-replica are in minority clusters, p is
inactive

### Full partitions versus subsets
As you can see above, in steady state, partitions are considered full (in that
they have all the relevant data). In some cases, for example in Figure 24 above
where an alternate replica of the partition p was created in Node 3, the
partition on node 3 is only a subset until all of the data in the partition copy
on Node 4 is synchronized with Node 3. Note that Node 4 has a full copy of
partition p since it split off from a fully available cluster. There are rules
on when and how to check for the second copy in order to ensure linearizability
versus sequential consistency. We will illustrate this using the following
scenario.

In a cluster with five nodes A, B, C, D, E, let us consider partition q that has
Node A as roster-master and Node B as roster-replica. Let us consider a rolling
upgrade where one node is taken down at a time. Initially Nodes A and B start
out as full partitions for q. When Node A is taken down, Node B which is
roster-replica promotes to alternate master for q and Node C becomes alternate
replica for q. Node C’s copy of partition q is now a subset. Soon enough Node A
rejoins the cluster (as subset) after the successful software upgrade and the
node B now goes down for its turn to be upgraded. At this point, there has not
been enough time for the roster-master A to complete synchronization of all its
data with B (that was Full). So, we are left with Node A as roster-master that
is a subset for partition q and also node C that is another subset for q. At
this point because this is a super cluster, we are guaranteed that among all the
nodes in the cluster, all updates to the partition are available. This is
because every update has to be written to at least two nodes (replication factor
2) and at most one node has been down at any one time. All changes must still be
in one of these nodes. However, what this means is that for all reads to records
that go to A (roster-master) every request has to resolve itself on a
record-by-record basis with the partition subset stored in node C. This will
temporarily create extra overhead for reads. Write overhead is never increased
as Aerospike writes to both copies all the time.

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_26.png)
{{/figure}}
 
Figure 26: Subset and Full Partitions during rolling upgrade process
So, the earlier rules are qualified further as follows:

1. If a sub cluster has all of the designated replicas (roster-master and
   roster-replicas) for a partition, and a full partition exists within the sub
   cluster then the partition is active for both reads and writes in that sub
   cluster
1. If a sub cluster has a majority of roster nodes and has either the
   roster-master or roster-replica for the partition within the component nodes
   and it has a full partition, then the partition is available 
1. If a sub cluster has exactly half of the nodes in the full cluster (roster)
   and it has the roster-master within its component nodes, and it has a full
   copy of the partition, then the partition is active for both reads and writes
1. If the sub cluster has a super majority (i.e., fewer nodes than replication
   factor are missing from the sub-cluster), then a combination of subset
   partitions are sufficient to make the partition active. 

Note there are some special kind of nodes that are excluded while counting the
majority and super-majority 
- A node with one or more empty drives or a brand-new node that has no data 
- A node that was not cleanly shutdown (unless commit-to-device) is enabled

Such nodes will have a special flag called “evade flag” set until they are
properly inducted into the cluster with all of the data

While we discussed the above using replication factor 2, the algorithm extends
to higher replication factors. All writes are written to every replica so the
write overhead will increase as replication factors increases beyond 2.

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_27.png)
{{/figure}}

Figure 27: Write logic

The write logic is shown in Figure 27. All writes are committed to every replica
before the system returns success to the client. In case one of the replica
writes fails, the master will ensure that the write is completed to the
appropriate number of replicas within the cluster (or sub cluster in case the
system has been compromised.)

### Strong Consistency for reads
In Aerospike strong consistency setting, reads are always sent to the master
partition. Note that the main invariant that the client software depends on is
that the server maintains the single master paradigm. However, Aerospike being a
distributed system, it is possible to have a period when multiple nodes think
they are master for a partition. Consider for example the case where node A of a
cluster is separated from the other four nodes B, C, D and E. B automatically
takes over for partition q and C becomes a new replica. Now, it is important to
differentiate the versions of the partitions where the writes are being done.
Note that the only successful writes are those made on replication-factor number
of nodes. Every other write is unsuccessful. Also, it is only possible for
exactly one sub cluster to take over as master for a partition based on rules
mentioned earlier. Even in this case, it is not possible to separate out the
writes that happen in a master overhang period by using just timestamps alone. 
So, Aerospike added a concept of regime for a partition. This regime is
incremented every time a master handoff for a partition happens. Only the old
master uses the earlier regime and all writes to the new master will use the
next regime. This means that changes made to a partition at a master node that
has not yet processed the cluster change can be discarded by comparing to a
greater regime number of the new sub cluster where the data is active. 

{{#figure "" "Aerospike Architecture" size="large"}}
![]({{book.assets}}/c_fig_28.png)
{{/figure}}

Figure 28: Clock that includes regime, last update time, and record generation

Aerospike uses the following per record as the mechanism to isolate the record
updates:
- 40 bits of record last update time (LUT)
- 6 bits of partition regime
- 10 bits of record generation

The 6 bits of regime provides about 27 seconds of buffer based on 1.8 seconds
for heartbeat intervals and accounts for around 32 cluster changes happening in
the period. The combination of regime and LUT and generation provides an
accurate path to determine which of the records in the system is the right value
for reading and writing.

### Linearize
Based on the above, in order to linearize reads at the server, every read to the
master partition needs to verify that the partition regimes are in sync for the
partition in which the key is located. If the regimes agree then the read is
guaranteed to be current. If the regimes do not agree this means that a cluster
change may be in process and it is important to redo/retry the read from the
client. Thus, for every write, all copies of the partition being written need to
also have the same regime.

### Session/Sequential Consistency 
In this case, the read from the master is all that is needed on the server-side. 
The Aerospike client (library) stores the partition regime, a 32-bit partition version
counter, as part of its partition table
based on the latest regime value it has encountered for a partition on its read.
This is to ensure that the Aerospike client (library) rejects any reads from servers of an older
regime than the one it has already read. This could happen due to an especially
large master overhang caused by slow system behavior or suspension/slowdown of
virtual machines in cloud environments, etc.

### Relaxed Consistency
There are also two Relaxed Consistency client policies: 'allow replica' and
'allow unavailable'. With the Relaxed Consistency modes, the client will
continue to read only committed records but reads will no longer be strictly
monotonic.

In practice, it is very difficult to encounter scenarios where the 'allow
replica' policy violates Session Consistency. This policy means applications
using Strong Consistency won't see read timeouts which may otherwise occur when
a node (or rack) goes down. Additionally, it enables clients to make use of the
'preferred rack' policy which is often used to reduce cross-zone network
utilization in various cloud environments.

The 'allow unavailable' policy relaxes consistency further by allowing clients
to read previously committed records on partitions marked as unavailable. This
mode allows applications which are sensitive to read unavailability to continue
to function during a major network/cluster disruption. When partitions are
available this policy behaves exactly like 'allow replica'.
