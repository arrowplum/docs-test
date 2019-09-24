---
title: ACID
description: Compatible with the latest in storage and database technology, Aerospike optimizes transaction throughput, latenct and uptime while guaranteeing as much correctness as possible.
assets: /docs/architecture/assets
---
The [CAP Theorem](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed) postulates that only two of three (consistency, availability, and partition-tolerance) properties can be guaranteed in a distributed system at any time. Since partitions are inevitable, and availability is paramount for a class of deployments, the Aerospike 3.x system prioritizes availability over consistency. The Aerospike system can therefore be classified as AP and works as follows to provide very high performance and availability:

- Prioritizing availability over consistency in each subsystem.
- Leveraging the Aerospike high vertical scale  (1 million TPS and multiple terabyte capacity per node) ensures small cluster sizes  (1 to 100 nodes).
- Ensuring low latency transactions at high throughput and keeping the data available during node failure and rolling upgrades.
- Providing some simple conflict resolution support to ensure that, during cluster change events, newer changes to data can "win" over older changes. 

In Aerospike 3, Aerospike solely supported Availability. In these cases, any partitioned set of servers would
claim complete ownership of all data. When cluster partitions were resolved, conflicts would occur,
causing data loss.

In Aerospike 4, Strong Consistency has been introduced. With this algorithm, cluster splits and partitions carefully
manage which section of the cluster is still available, disallowing any potentially conflicting writes.

Aerospike 4 maintains consistency on a primary key basis. It provides Durability by committing the writes to multiple
physical servers with independent hardware components. For even greater durability, a write may be required to flush
to persistent storage.

ACID, however, typically also implies two further features:
- Multi-record isolation and consistency
- Query isolation, so a multi-record query will be consistent in the face of writes

Aerospike does not provide these two features.

Please read the [Strong Consistency Feature Guide](/docs/guide/consistency.html) for further information.
