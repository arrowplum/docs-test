---
title: Secondary Index
description: In Aerospike, the secondary key index provides predictable and fast access to multiple rows of information in a database.
assets: /docs/architecture/assets
---

Secondary indexes are on a non-primary key, which allows you to model one-to-many relationships. In Aerospike, secondary indexes are specified on a bin-by-bin basis (like RDBMS *columns*). This allows efficient updates and minimizes the amount of resources required to store indexes. 

A data description (DDL) determines which bins and data types to index. Use the Aerospike tools or the API to dynamically create and remove indexes. This DDL (similar to RDBMS schema) is **not** used for data validation&mdash;even if a bin is in the DDL as indexed, that bin is optional. For an indexed bin, updating the record to include the bin updates the index.

Index entries are type checked i.e If you have a bin that stores user age, and the age value is stored as a string by one application and an integer by another application, an integer index excludes records containing string values stored in the indexed bin, while a string index excludes records with integer values stored in the indexed bin.

Secondary indexes: 

- are stored in RAM for fast look-up.
- are built on every node in cluster and co-located with the primary index.
 Each secondary index entry only contains references to records local to the node.
- contain pointers to both master records and replicated records in the node.

### Data Structure

<div style="float: left" >
{{#figure "" "Secondary index with B-tree" size="large"}}
![]({{book.assets}}/data_structure_small.png)
{{/figure}}
</div>

This example illustrates that like the Aerospike primary index, the secondary index is blend of a hash table with a B-tree. The secondary index is structurally a hash of *B-tree of B-trees*. Each logical secondary index has 32 physical trees. For index operation on key identification, a hash function determines which physical tree for the secondary index entry must be made out of 32 physical 
trees. After identificaiton, B-tree updates are made into the tree. Note that corresponding to the single secondary index key, there
can be many primary record references. The lowest level of this structure is a list of full-fledged B-trees based on the number of records that need to be stored. 
<br />
<br />
<br />
<br />
### Index Management

#### Index Metadata 

<div style="float: right" >
{{#figure "" "Secondary indexes triggered from SMD" size="large"}}
![]({{book.assets}}/index_metadata_small.png)
{{/figure}}
</div>

Aerospike tracks which indexes are created in a globally maintained data structure&mdash;the System Metadata (SMD) system. The SMD module resides in the middle of multiple secondary index modules on multiple nodes. Changes made to the secondary index are always triggerred from the SMD. 

SMD workflow is:

1. A client request triggers a create, delete, or update related to the secondary index metadata. 
 The request passes through the secondary index module to the SMD. 
1. SMD sends the request to the paxos master.
1. The paxos master requests relevant metadata from all cluster nodes. 
1. Once all the data is received, it calls the secondary index merge callback function. 
 This function resolves the winning metadata version for the secondary index. 
1. SMD sends a request to all cluster nodes to accept the new metadata.
1. Each node performs a secondary index create or delete DDL function.
1. A scan is triggered and returns to the client.

#### Index Creation

Aerospike supports dynamic creation of secondary indexes. Tools (such as [aql](/docs/tools/aql)) are available to read the current indexes and allow creation and destruction of indexes. 

To build a secondary index, you specify a namespace, set, bin, container type (none, list, map), and data type (integer, string, [Geospatial](/docs/guide/geospatial.html#geospatial-index), and so on). On confirmation by the **SMD** (see above), each node creates the secondary index in write-active mode, and starts a background scan to scan all data and insert entries in the secondary index. 

- Index entries are only created for records matching all of index specifications.
- The scan populates the secondary index and interacts with read/write transactions exactly as a normal scan, except that there is no network component to the index creation. 
During index creation, all new writes affecting the indexing attribute update the index. As soon as the index creation scan completes and all index entries are created on all cluster nodes, the index is marked read-active and is ready for use by queries.
- Aerospike also supports indexing on top level list and maps

**Recommendations**
- Avoid index DDL (create-drop index) when the cluster is not well formed or if it is experiencing integrity faults. 
 Index building is a heavy I/O subsystem operation, so should only be done during low load.
- If a node with data joins the cluster but has missing index definitions, indexes are created and populated after it joins the cluster.
 During index population, queries are not allowed to ensure that data on the incoming node is clean before it is available.

##### Create Index Priority

The index creation scan only reads records already committed by transactions (that is, no dirty reads are allowed). This means that scans can execute at full speed, provided there are no updates for records to block reads. 

{{#note}}
To ensure that the index creation scan does not adversely affect the latencies of ongoing read and write transactions, set the index build priority to the proper level. Use the job prioritization settings in the Aerospike realtime engine to control resource utilization for the index creation scan. 
{{/note}}

Aerospike leverages years of deployment experience to ensure that the default settings suffice, since they are based on balancing long running tasks (such as data rebalancing and backup) against low-latency read/write transactions.

### Writing Data with Indexes

On data writes, the SMD specification of current indexes is checked. For all bins with indexes, a secondary index update-insert-delete operation is performed. Note that Aerospike is a flexschema system. If no index value exsists on a particular bin or if the bin value is not a supported index type, then the corresponding secondary action is not performed.

All changes to the secondary index are performed atomically with the record changes under single-lock sychronization. Since indexes do not persist, difficult commit problems of committing the index and the data are removed, which increases access speed.

### Garbage Collection

On data deletes (for example, during delete, expiry, eviction, or migration operations), the data is not read from disk to delete the 
entry from the secondary index. This avoids unnecesarry burden on the I/O subsystem. The remaining entries in the secondary index are deleted by a background thread, which wakes up at the regular intervals and performs cleanup. 

This **Garbage Collector** is designed to be non-intrusive. It creates a list of entries to remove in small batches, and then slowly deletes them from the index. This requires provisioning of memory for the secondary index in systems with a high expiriration and eviction rates.

### Distributed Queries

This illustrates an index architecture with distributed queries. 

![Secondary Indexes]({{book.assets}}/query_diagram.png)

Every cluster node recieves the query to retrieve results from the secondary index. When the query executes:
1. Requests “scatter”  to all nodes.
1. Indexes are placed in RAM for fast mapping of secondary-to-primary keys.
1. Indexes are co-located on each node with data on SSDs to provide fast update performance.
1. Records are read in parallel from all SSDs and RAM.
1. Results are aggregated on each node.
1. Results are “gathered” from all nodes and returned to the client.

A secondary index query can evaluate a very long list of primary key records. This is why Aerospike performs secondary index queries in small batches. Batching also occurs on client responses, so that if a memory threshold is reached, the response is immediately flushed to the network, much like return values in an Aerospike batch request. This keeps memory usage of an individual secondary query to a constant size, regardless of selectivity. 

#### Query Result 

The query process ensures that results sync with actual data every time the query executs and the record is scanned. Uncommitted data reads are never part of the query results. 

##### Cluster State Changes

Secondary index rebuilds and query results consistency scenarios are listed below.

| Scenario | Persistent Namespace Boot Type | Data-in-memory | Secondary Index Population | Node Boot Time | Query Consistency during migrations |
| --- | --- | --- | --- | --- | --- |
| Node joining | With data; without the fast restart feature enabled (1)| False | Post data load from disk; parallel data partition scan | Longer data load times than with no secondary index | Best effort (2) | 
| Node joining | With data; with the fast restart feature enabled (primary key index is available in shared memory) | False | Post fast restart; parallel data partition scan (1) | Longer data load time than with no secondary index | Best effort (2) | 
| Node joining | With data; the fast restart feature enabled | True | At data load from disk | No signficant difference with and without a secondary index | Best effort (2) | 
| Node joining | No data; Always without the fast restart feature enabled | True/False | -NA- | -NA- | Consistent copy | 
| Node leaving | - NA - | True/False | -NA- | -NA- | Consistent copy | 

1. Fast restart is not supported for secondary indexes.
2. Best Effort is a transactionally consistent copy of data, but not necessarily latest. With duplicate records, merge is **not** performed before returning results by the secondary index query. After all migrations complete, there is always a consistent copy.

In a normal operating environment, nodes are simply added and removed from the cluster maintaining complete query availability. Aerospike  loads the data from disk during migrations.

##### Query Execution During Migrations

Getting accurate query results is complicated during data migrations. When a cluster node is added or removed, it invokes the **Data Migration Module** to transition data to and from nodes as appropriate for the new configuration. During the migration operation, a partition may be available in different versions on many nodes. For a query to locate a partition with the requested data, Aerospike query processing uses additional partition states shared among cluster nodes, and selects a node for each partition where the query can execute. The node can be the master node of the partition, the old master, or replica node that is migrating data to the new master node. Duplicate resolution is not performed, even if there are multiple versions of the data.

### Aggregation  

Query records can feed into the aggregation framework to perform filtering, aggregation, and so on. Each node sends the query result to the UDF (user-defined function) sub-system to start results processing as a stream of records. [Stream UDFs](/docs/architecture/udf.html#stream-udf) are invoked and the sequence of operations defined by the user are applied to the query results. Results from each node are collected by the client application, which can then perform additional operations on the data.

#### Performance

To ensure that aggregation does not affect overall database performance, Aerospike uses these techniques:

- Global queues manage records fed through the different processing stages, and thread pools effectively utilize CPU parallelism.
- The query state is shared across the entire thread pool so that the system can manage the Stream UDF pipeline.

{{#note}}
Except for the initial data fetch portion, every stage in aggregation is a CPU-bound operation. It is important that processes finish quickly and optimally. To facilitate this, Aerospike will batch records, cache UDF states, and so on to optimize system overhead.
{{/note}}

- For namespace operations where data is stored in-memory and no storage fetch is required, Aerospike implements stream processing in a single thread context.
 Even with this optimization, the system can parallelize operations across data partitions because Aerospike natively divides data as a fixed number of partitions.
