---
title: Distribution
description: The Aerospike Database handles big data with 24/7 availability. 
assets: /docs/architecture/assets
---

We designed the Aerospike Database for applications that must be available 24/7 and handle big data with reliability. This has several implications:

- Automatic data location detection 
 There is no need to worry about data location. The Aerospike client automatically detects data location and ensures that requests are handled in a single hop. Your application can treat the database as if were stored on a single server. The Aerospike **smart client** handles cluster distribution.
- Automatic cluster balancing
 When adding capacity, simply add a node to the cluster and the cluster re-balances to include the new node. Throughput and 
performance linearly scale as you add capacity.
- No single point of failure
 An SSD on a node can fail, a node can fail or be taken offline for maintenance or upgrades, or the entire datacenter can fail without affecting reliability.

Managing the cluster reliably is the core of the Aerospike Database, and is taken very seriously. Aerospike achieves this using the following features:

- **[Data Distribution](/docs/architecture/data-distribution.html):** Robust partitioning ensures uniform data distribution, which 
avoids hot spots and automatically balances data without manual intervention.
- **[Clustering](/docs/architecture/clustering.html):** The Aerospike clustered database automatically detects failures and heals.
- **Replication:** This Aerospike feature includes the following replication abilities to avoid a single point of failure:
 - **Intra Cluster Replication** 
 - **[Rack Aware Replication](/docs/architecture/rack-aware.html)**
 - **[Cross-Datacenter Replication](/docs/architecture/xdr.html)**

