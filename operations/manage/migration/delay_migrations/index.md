---
title : Delaying "Fill" Migrations
---


When a node is removed from a cluster, other nodes will stand in to hold replicas (masters or proles) that the missing node held.

The stand-in replicas will be filled via migrations from the other replica(s) remaining in the cluster.  Write transactions will also replicate to the stand-in replicas.  While these replications are essential to maintaining consistency, the filling of stand-in nodes via migration is not essential.

The only reason to do such "fill" migrations is to prepare the cluster for a (long-term) future in which the missing node remains out of the cluster, and the stand-in replicas become "normal" replicas.  If the missing node returns to the cluster, the stand-in replicas will ultimately be dropped.

Therefore in situations where a node will only be out of the cluster for a short time (e.g. during rolling upgrades), it may be better to not bother doing these "fill" migrations at all -- avoiding them will save on background processing, device IO (if data not in memory), and data movement.  It may also save on temporary memory and storage consumption which may otherwise be hazardous in certain scenarios, e.g. rack-aware clusters with very few racks.

The (dynamic) configuration item [`migrate-fill-delay`](/docs/reference/configuration/index.html#migrate-fill-delay) (Enterprise Edition only) can be used to suppress "fill" migrations.

### Defining "Fill" Migrations

A "fill" migration is one which is filling a node that is not normally a replica.

In SC ([`strong-consistency`](/docs/reference/configuration/index.html#strong-consistency)) enabled namespaces, this is easy to identify -- if the node is not a "roster replica" for the partition in question, any migration to that node is deemed a "fill" migration.  The roster allows us to know which nodes are intended to be in the cluster, and (per partition) which nodes hold the "normal" replicas.

In AP (non SC) clusters, we have no such guidance as to what nodes are supposed to be in the cluster.  Instead we use the heuristic that a migration to any node which is empty (or more exactly, started out empty and hasn't been filled) is a "fill" migration.

This means that there are ordinary scenarios in AP clusters where "fill" migrations should not be suppressed.  E.g. if a new empty node is added and becomes a replica for a given partition, we would want the node to be filled.

Note that the same situation in an SC cluster would be fine with "fill" migrations suppressed, since the new node would have been added to the roster, would be identified as a "roster replica", and therefore the migration would not be deemed a "fill" migration, even though it will fill the new node.

Operators must be aware of their situation in order to choose how to set up "fill" migrations.

### Configuring [migrate-fill-delay](/docs/reference/configuration/index.html#migrate-fill-delay)

[`migrate-fill-delay`](/docs/reference/configuration/index.html#migrate-fill-delay) is a dynamic service context configuration item with units of seconds.  It can be configured with 'm', 'h', or 'd' for minutes, hours, or days, e.g. 1h instead of 3600.  The default is 0, meaning "fill" migrations are not delayed.  A non-zero value does the obvious -- delays beginning "fill" migrations for the specified amount of time, measured from the rebalance that scheduled them.

{{#note}}
For versions 4.5.0.2 and earlier, using time units ('m', 'h' or 'd') does not work when setting this configuration parameter dynamically.
{{/note}}

Changing the config value takes effect within a second.  E.g. if it had been set to 5 minutes, and only one minute has elapsed since rebalance, changing it to 0 (or anything less than a minute) will cause the delayed "fill" migrations to start right away.

### Possible Scenario

For a cluster where nodes can be upgraded and [fast restarted](/docs/operations/manage/aerospike/fast_start/index.html) in approximatively 2 minutes: configure [`migrate-fill-delay`](/docs/reference/configuration/index.html#migrate-fill-delay) to be 5 minutes.

Rolling upgrades should then see no fill migrations at all -- when a node is removed, the fill migrations will not start, and the node should be back before they would have started.  All migrations on the node's return will be "delta" migrations -- not "fill" migrations -- and will proceed immediately.

For an SC cluster one may consider leaving [`migrate-fill-delay`](/docs/reference/configuration/index.html#migrate-fill-delay) configured this way normally, and only change it if there is an unusual scenario in which it is desired for fill migrations to begin immediately.  Note that even if a node goes down unexpectedly, after 5 minutes a stand-in node would still be filled, so that when what happened is later discovered, if the roster is to be changed to permanently remove the downed node, a new roster replica already full is ready.