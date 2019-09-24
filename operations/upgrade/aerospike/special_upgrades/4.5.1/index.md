---
title: Upgrading Aerospike to 4.5.1+
description: Learn the proper procedures to ensure there is no downtime required to upgrade from previous Aerospike versions.
styles:
  - /assets/styles/ui/steps.css
---


{{#warn}}
If planning to run a mixed cluster of 4.5.1+ and pre-4.5.1 nodes for a prolonged period with expiration occurring please upgrade the pre-4.5.1 nodes to the [Aerospike Server version 4.5.0.10](/download/server/notes.html#4.5.0.10) release.
For more information please refer to the section [Preventing Replica Build Up](#preventing-replica-build-up).
{{/warn}}

## Expected Warnings to be Ignored:
The following may be observed on older nodes when 4.5.1+ nodes have been added
to the cluster.

```
WARNING (smd): (system_metadata.c:2152) Null response message passed in transaction complete!
```

The following may be observed on 4.5.1+ nodes when there are still pre-4.5.1 node
present in the cluster.

```
WARNING (smd): (smd.c:756) failed to parse msg op_type 0
```

## Who is Affected?
Special considerations may apply if your deployment has used any of the
following features:

- expirations
- evictions
- strong-consistency
- security
- secondary indexes
- truncate
- truncate-namespace
- UDFs (User Defined Functions)

If your deployment doesn't use any of these features, then aside from a few
warnings already mentioned there shouldn't be any concerns for this upgrade.
However, if your deployment uses eviction (possible if records are allowed to
expire) then read the section on ["General Upgrade Pitfalls"](#general-upgrade-pitfalls-)
because future upgrades will also be affected.

## General Upgrade Pitfalls:
While running with a mixed cluster of 4.5.1+ and pre-4.5.1 nodes, SMD (System
MetaData) update requests will not propagate if the request arrives at pre-4.5.1
node. Requests arriving at a 4.5.1+ node will only propagate to other 4.5.1+
nodes within the cluster. 

Expirations and evictions would also be impacted when running in a mix environment.
On version 4.5.1 and above nodes would expire and evict both master and replica objects locally. On versions prior to 4.5.1, only master objects would get evicted or expired locally on the node, and a delete transaction would be internally generated in order to delete the relevant replica objects. In a cluster running mixed versions (prior to 4.5.1 as well as above), the nodes running version 4.5.1 and above will not issue delete transactions to nodes holding the replica records (SMD protocol version not compatible) to any of the nodes running versions earlier than 4.5.1, resulting in replicas not being deleted on these nodes. On the other side, the nodes running version 4.5.1 and above would receive delete transactions for the replica objects. This could result in an imbalance of master/replica objects in such clusters.

The following operations will be unreliable while running
in a mixed cluster:

- Proper expirations and evictions across replica records.
- Adding or modifying namespace rosters.
- Adding, removing, or modifying security definitions.
- Adding, removing, or modifying secondary index definitions.
- Truncate and truncate-undo info commands.
- Truncate-namespace and truncate-namespace-undo info commands.
- Adding, removing, or modifying UDF definitions.

## Preventing Replica Build Up

There is a special configuration item 'extra-prole-ttl' unique to the Aerospike Server version 4.5.0.10 release, to support running a mixed cluster of 4.5.1+ and pre-4.5.1 nodes for a prolonged period with expiration occurring without building up expired replica objects on the older nodes.

If the service context 'prole-extra-ttl' configuration item is set to a non-zero value, it activates garbage collection of expired replica records, and specifies the number of seconds beyond a replica record's expiration time that the record becomes eligible to be deleted by this process.

By default the setting is zero. It can be set in the configuration file (aerospike.conf), or changed dynamically:

```
asinfo -v "set-config:context=service;prole-extra-ttl=120"
```

The above example would make replica records that have aged 2 minutes beyond their ttl eligible for removal.

At the end of each nsup cycle, if any replica records were deleted by this process, there will be an info log line in the server log file showing the number of deleted records.

```
expired-non-masters 1234
```

## Docker/Kubernetes/etc. 'Cloud-Style' Upgrade Pitfalls:
Since SMD in 4.5.1+ isn't network protocol compatible with pre-4.5.1 versions,
Aerospike relies on the contents of `/opt/aerospike/smd` and
`/opt/aerospike/sys` to be persisted. In some cloud-style upgrade processes,
these directories are not persisted.

In general, for any upgrade these directories should be persisted, especially if
truncation (or, for 4.5.1+, eviction) features are used. These features use
SMD-persisted items during startup and so cannot rely on receiving SMD items via
the network.

In this particular upgrade, since SMD items will not replicate via network from
pre-4.5.1 nodes, it is especially important to ensure all SMD directories are
persisted.
