---
title: Hybrid Storage
description: In Aerospike, the Hybrid Memory System contains indexes and data stored in each node, handles interaction with the physical storage, and contains modules for automatically removing old data from the database. 
assets: /docs/architecture/assets
---
The Hybrid Memory System contains indexes and data stored in each node, handles interaction with the physical storage, contains modules for automatically removing old data from the database, and defragments physical storage to optimize disk usage.

###Storage Media

Aerospike can store data in DRAM, SSDs, and traditional spinning media. Each *namespace* is separately configurable, which allows developers to put small, frequently accessed namespaces in DRAM, and put larger namespaces in less expensive storage such as SSD. Aerospike optimizes data storage on SSDs by bypassing the file system, which takes advantage of low-level SSD read and write patterns.

In Aerospike:

- Record data is stored together. 
- The default storage size for one row is 1 MB. 
- Storage is copy-on-write.
- Free space is reclaimed during defragmentation. 
- Each *namespace* has a fixed amount of storage, and each node must have the same namespaces on each server, which requires the same amount of storage for each namespace.

It is best to configure storage with pure DRAM without persistance, DRAM with storage persistance, or using Flash storage (SSDs). Persistant storage (that is, disks) must be Flash or another high-performance block storage device such as cloud storage. Persistent storage can also be a file on any storage device.

### DRAM

Pure DRAM storage&mdash;without persistance&mdash;provides higher throughput. Although modern Flash storage is very high performance, DRAM has better performance at a much higher price point ( especially including the cost of power ).

Aerospike allocates data using *JEMalloc*, allows allocation into different pools. Long-term allocation such as for the storage layer, is separately allocatable. JEMalloc has exceptionally low fragmentation properties.

Aerospike achieves high reliabliltiy by using multiple copies of DRAM. Since Aerospike automatically reshards and replicates data on failure or during cluster node management, _k-safety_ is obtained at a high level. When a node returns online, its data automatically populates from a copy.

Aerospike uses random [data distribution](/docs/architecture/data-distribution.html) to keep data unavailbility when several nodes are lost very small. In this example, we have in a 10-node cluster with two copies of the data. If two nodes are simultaneously lost, the amount of unavailable data before replication is approximately 2% or 1/50th of the data. With a persistant storage layer, reads always occur from a copy in  DRAM. Writes occur through the data path described below.

### Data on SSD/Flash

When a write ( update or insert ) has been received from the client, a latch is taken on the row to avoid two conflicting writes on the same record for this cluster ( in the case of network partition, conflicting writes may be taken to provide availability, which are resolved later ). In some cluster states, data may also need to be read from other nodes and conflicts resolved. After write validation, in-memory representations of the record update on the master. The data to be written to a device is placed in a buffer for writing. When the write buffer is full, it is queued to disk. Write buffer size (which is the same as the maximum row size) and write throughput determine the risk of uncommitted data, and configuration parameters allow flushing of these buffers to limit potential data loss. Replicas and their in-memory indexes then update. After all in-memory copies update, the result returns to the client. 

### Storing Data

Aerospike supports the following data types:

- integers
- doubles
- strings
- blobs
- lists
- maps
- geoJSON
- natively serialized types

Columns in Aeropsike each have a bin name stored using a string table. The name of the column is stored only once, and only 32K concurrent unique bin names are allowed in a namespace (see also [Single Bin Optimization](/docs/architecture/primary-index.html#single-bin-optimization)).

If you require more bin names, use maps. Map allow you to store an arbitrary set of key-value pairs, and access those values in UDFs for efficiency.

If you pass a complex language type (such as your own class in Java) into a data call, the Aerospike client uses the native serialization system. Aerospike stores that data as a _blob_ type specific to the language. This allows a client using the same language to read data with clean code. But most default serializers are inefficient.

Integers are stored as 8-byte quantities, which limits integer values in the current version. The Aerospike network protocol allows integers of variable size, which allows expansion. Aerospike stores strings as UTF-8. UTF-8 is more compact than unicode for many strings. To allow cross-language compatibility, client libraries convert the native character unicode to UTF-8, and back.

Binary objects (blobs) are most efficient in that they are size-limited only by the size of the total record. Some customers will use their own serializer, and possibly compress the object and store it directly. This means that data cannot be easily accessed using UDFs, nor can atomic operations for List and Map be used. Aerospike's [storage compression](/docs/operations/configure/namespace/storage/compression.html) feature avoids this side-effect of application-side compression.

Complex data types are rendered to `msgpack` for local storage. Complex objects are serialized on the client, and sent using wire protocol. When used for simple `get()` and `put()` operations, network format is written directly to storage without serialization or conversion.

### Flash Optimization

The Aeropsike **Defragmenter** tracks the number of active records on each block on disk, and reclaims blocks that fall below a minimum level of use. The defragmentation system continually scans available blocks, and looks for blocks with a certain amount of free space. 

### Eviction Based on Storage

The Aerospike **Evictor** removes references to expired records, and reclaims memory when the system gets beyond a set high-water mark. When configuring a namespace, specify the maximum amount of DRAM avaliable to it and the default time-to-live (TTL) or expiration for its data. 

The Evictor searches for expired data, frees the in-memory index, and releases the record on disk. It also tracks memory used by the namespace and if memory exceeds the set high-water mark, releases older although not necessarily expired records. By allowing the Evictor to remove old data when the system hits memory limitations, Aerospike can effectively be used as an LRU cache.
Note that the age of a record is measured from the last time it was modified, and that the Application can override the default lifetime any time it writes data to the record. The Application may also tell the system that a particular
record should never be automatically evicted.
