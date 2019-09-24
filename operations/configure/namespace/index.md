---
title: Namespace Configuration
description: Aerospike's namespaces can be configured and tuned to fit a wide variety of use cases. Play with recipes for the most common namespace configuration variations. 
---

Aerospike's namespaces can be configured and tuned to fit a wide variety of use
cases. In this section we will compare and provide configuration recipes for
the most common namespace configuration variants. For a complete listing of
namespace parameters see the
[configuration reference](/docs/reference/configuration).

The minimum requirement for configuring a namespace is to provide a
namespace-name. This will create an in memory namespace by that name with a 4GB
capacity. The configuration would be similar to the following, the commented
parameters indicate the defaults.

```bash
namespace <namespace-name> {
    # memory-size 4G           # 4GB of memory to be used for index and data
    # replication-factor 2     # For multiple nodes, keep 2 copies of the data
    # high-water-memory-pct 60 # Evict non-zero TTL data if capacity exceeds
                               # 60% of 4GB
    # stop-writes-pct 90       # Stop writes if capacity exceeds 90% of 4GB
    # default-ttl 0            # Writes from client that do not provide a TTL
                               # will default to 0 or never expire
    # storage-engine memory    # Store data in memory only
}
```
In the following sections we will dive deeper into namespace and storage
configuration and provide configuration recipes to enable you to get started
quickly.

{{#warn}}
For versions not running the latest cluster protocol ([introduced in versions 3.13](/docs/operations/upgrade/cluster_to_3_13/index.html)), 
adding or removing a namespace requires a full cluster shutdown. It is also critical on these older versions of the server  to keep the same namespaces order in aerospike.conf throughout the cluster.
{{/warn}}

{{#warn}}
There is a maximum limit of 32 namespaces in a cluster in the Aerospike Enterprise Edition Server version and 2 namespaces in a cluster in the Aerospike Community Edition Server version (as of version 4.0).
{{/warn}}

### Where to Next?
- Configure [Storage Engine](/docs/operations/configure/namespace/storage) which determines if and where records are
  persisted to.
- Configure [Data Retention Policy](/docs/operations/configure/namespace/retention) which determines how long
  records are kept after it is initially written.
- Configure [Data Durability Policy](/docs/operations/configure/namespace/durability) which determines how many
  replica copies of a record to keep in the cluster.
- Or return to [Configure Page](/docs/operations/configure).
