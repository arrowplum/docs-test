---
title: Namespace Durability Configuration
description: Find out how you can configure Aerospike to provide intracluster replication and cross cluster replication to protect your data from server failure.
---

An important consideration when configuring an Aerospike namespace
is how durable does the data need to be. Aerospike provides intracluster
replication and cross cluster replication to protect data from server
failure.

### Intracluster Replication
In Aerospike, the `replication-factor` configuration controls the number of copies of each
record that reside in the cluster. Most servers have a `replication-factor` of 2,
which ensures all data in the cluster can survive a single node failure.
Replication does increase the total cost of a cluster. With a replication factor
2, the cluster requires twice the storage capacity of a cluster with a replication factor of 1. Replication also suffers a performance penalty, primarily due to the
extra network latency incurred by synchronous replication.
{{#info}}
Aerospike replicates synchronously on writes to ensure that all copies are
consistent across the cluster, which can impact performance. See the
[XDR documentation](/docs/operations/configure/cross-datacenter) for instructions on asynchronous
cross-datacenter replication.
{{/info}}

Configuring the replication factor is simple. The following example shows a replication factor of 2:

```
namespace <namespace-name> {
	...
	replication-factor 2
	...
}
```

### Cross Cluster Replication
Aerospike Enterprise Edition includes the Cross Datacenter Replication (XDR)
product to replicate data between clusters.
For more information on XDR configuration, see the [configuration documentation](/docs/operations/configure/cross-datacenter).

### Where to Next?
- Configure [Storage Engine](/docs/operations/configure/namespace/storage) which determines if and where records are
  persisted.
- Configure [Data Retention Policy](/docs/operations/configure/namespace/retention), which determines how long
  records are kept.
- Or return to [Configure Page](/docs/operations/configure).
