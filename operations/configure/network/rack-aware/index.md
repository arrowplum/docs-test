---
title: Rack Aware Configuration
description: Configure rack awareness
---

In 3.13, Aerospike has greatly simplified operational ease-of-use for rack-aware deployments.
If you have a deployment older than version 3.13 and would like to switch to rackaware configuration, we recommend you first [upgrade to 3.13](/docs/operations/upgrade/cluster_to_3_13). If you must remain on the older version to configure rackaware, please see section "Version Earlier than 3.13" for instructions.

{{#warn}}
The rack-aware feature is an Aerospike Enterprise Edition Server only feature as of version 4.0.
{{/warn}}

## Configure a New Rackaware Cluster
Rackaware can be specified at the namespace level. For nodes which should be on the same rack, simply specify the same rack-id for these nodes -

```
namespace {
    ...
    rack-id 1
    ...
}
```

Rack-id must be an integer between 1 and 1000000. 

## Upgrade a Cluster for Rack Awareness
Rackaware configuration for an existing cluster can be enabled or updated either through a rolling restart or dynamically (for versions 3.13 and above, which are running the new cluster protocol).

### Dynamic
- on each node change the rack-id to the desired value using the asinfo command: 
```bash
asinfo -v "set-config:context=namespace;id=namespaceName;rack-id=1"
```
- add the [`rack-id`](https://www.aerospike.com/docs/reference/configuration/#rack-id) configuration parameter to the namespace stanza in the config file to ensure the configuration remains following any future restarts.
- trigger a rebalance of the cluster to engage migrations with the new rack-id configurations:
  - for aerospike version 3.14.1.1 and above, issue the recluster command: `asadm -e "asinfo -v recluster:"`
  - alternatively, for aerospike versions 3.13 running the new cluster protocol, restart one of the nodes to trigger migrations (there is no need for a full rolling restart of all the nodes).

### Rolling restart method
- Take a node down.
- Add [`rack-id`](/docs/reference/configuration/#rack-id) to the desired namespace(s).
- Bring the node back up.
- Repeat for next node, until all nodes/namespaces have the desired [`rack-id`](/docs/reference/configuration/#rack-id).

{{#note}}
**Note** - Make sure clusters have adequate capacity when using the rolling restart method. When the first node is restarted it will have a different [`rack-id`](/docs/reference/configuration/#rack-id) than all other nodes within the cluster and so will receive a whole rack worth of traffic and attempt to keep a copy of every partition for any rack aware namespaces. It is recommended to use the dynamic method.
{{/note}}

## Display Rack Group Settings
To get the grouping of the racks -

```bash
asinfo -v "racks:""
ns=test:rack_1=BCD10DFA9290C00,BB910DFA9290C00:rack_2=BD710DFA9290C00,BC310DFA9290C00
```
For the example above, for the `test` namespace, rack-id 1 includes nodes BCD10DFA9290C00,BB910DFA9290C00, rack-id 2 includes nodes BD710DFA9290C00,BC310DFA9290C00

## Version Earlier than 3.13

### Configure a New Rackaware Cluster
To configure rack awareness, the following needs to be done:

- Configure the rack-aware protocol.
- Configure each server node with its group.

These configuration changes are described in detail below.

#### Configuring the Rack-Awareness Protocol

Update the paxos-protocol parameter in the service stanza to be v4 as shown below:

```
service {
    ...
    paxos-protocol v4
    ...
}
```

#### Configuring the Group (Rack)

In the non-rack-aware case, each node is identified by a node-value -- a 64-bit unsigned integer which consists of: 16-bit port ID plus the 48-bit hardware MAC address.  Each node-value in the cluster must be unique.

When using rack awareness, a node-value consists of:  16-bit port ID + 16-bit group ID + 32-bit node ID:

- The 32-bit node ID can be automatically generated from the node's IP address OR you can specify it explicitly. Specifying the node ID explicitly might be used when you want more control over the node ID  value for verification or cluster debugging.
- The group ID must be specified explicitly.

**Explicit Node ID**

Set each node's ID and group by inserting the following top level stanza into a node's configuration file:

```
cluster {
    mode static
    self-node-id [32-bit unsigned integer node ID]
    self-group-id [16-bit unsigned integer group ID]
}
```

**Auto Generate Node ID**

Inserting the following top level stanza into each node's configuration file: (the node ID is computed based on IP address)

```
cluster {
    mode dynamic
    self-group-id [16-bit unsigned integer group ID]
}
```

**Caution:**

When you choose the dynamic option and use the IP address as part of the
node ID, there is some additional bookkeeping needed.
You must take care to avoid reusing IP addresses for nodes in a cluster, since
that affects the uniqueness of the node iDs.
The node IDs of different database nodes must never conflict; such a conflict
would result in the cluster not forming correctly.

The node ID for a database node is computed once, at the start of the node's
lifetime in a cluster instance, so even though that node's actual IP address
may change over time, its node ID is fixed for either its lifetime or the
lifetime of the cluster (whichever is shorter).
Therefore, even though a cluster's node IP addresses at any one time may
not conflict,
in order to keep them guaranteed unique over the lifetime of the cluster, no
active IP address may overlap with any previously started node IP address.

**Notes:**

1. If the node-values (port+group+node) are not unique within a cluster then
the nodes with duplicate node-values will not be able to join the cluster.
2. The node ID and group ID must be positive (non-zero) integers.
3. To turn off rack-aware support, users can comment out or delete the "mode"
line, or they can use the value "none":  e.g. mode none.
This will restore default (non-rack-aware) cluster behavior.
4. A cluster in rack-aware mode can have a mixture of nodes with auto-generated
and explicit node IDs – that is, you can mix and match how node IDs are
specified.

For example, to configure two nodes into a group, the first node might have the following in its configuration file:

```
service {
    ...
	paxos-protocol v4
	...
}
 
cluster {
    mode static
    self-node-id  101
    self-group-id 201
}
```

and the second node might have the following in its configuration file:

```
service {
    ...
	paxos-protocol v4
	...
}

cluster {
    mode dynamic
    self-group-id 201
}
```

On startup, when a node joins the cluster, the existing nodes in the cluster will discover the group topology and assign replica partitions accordingly.

### Upgrade a Cluster to Rackaware

To upgrade your cluster to use rack awareness:

1. Stop all cluster nodes.
1. Upgrade their configuration files, similar to bringing up new cluster.
1. Restart the cluster. 
 - Since replica locations must be agreed upon by all cluster nodes, when you first enable rack awareness you must restart the entire cluster.

If you enable rack awareness, all nodes in the cluster must use rack awareness; that is, configure all nodes as described above. Rack awareness requires a minimum of two groups. If there is only one node or only one group, then the default (non-rack aware) behavior automatically takes over. This is also the case if multiple groups (racks) are present on cluster start, but failure causes only a single group (rack) to be up. The remaining nodes in the single remaining group (rack) automatically form a default (non-rack aware) cluster.

### Scale a Rackware Cluster

Once rack awareness is enabled, you can add a node to a group or remove a node from a group without restarting the whole cluster. To add a node to the cluster, simply configure the group before starting the node. The cluster rebalances.

Node values (that is, the 64-bit identifier comprised of port+group+node) _MUST_ be unique. When you first enable rack awareness, this uniqueness is easy to configure; however, over hardware upgrades and IP addresses changes, uniqueness may be harder to maintain, especially with a mix of static and dynamic node values. It is a best practice to create and carefully maintain a list of current node values. When a new machine enters the cluster and there is a node-value collision, odd cluster behavior may occur because the node communication has faults.

### Display Rack Group Settings

To see the nodes in a group:

```bash
asinfo -v dump-ra:verbose=true
```

In this 3-node/3-group cluster example, the output of a query using node ID 101 is:

```bash
May 28 2013 18:39:00 GMT: INFO (info): (base/cluster_config.c:267) Rack Aware is enabled.  Mode: static.
May 28 2013 18:39:00 GMT: INFO (paxos): (base/cluster_config.c:281) SuccessionList[0]: Node bcd00cb00000069 : Port 3021 ; GroupID 203 ; NodeID 105 [Master]
May 28 2013 18:39:00 GMT: INFO (paxos): (base/cluster_config.c:281) SuccessionList[1]: Node bc300ca00000067 : Port 3011 ; GroupID 202 ; NodeID 103
May 28 2013 18:39:00 GMT: INFO (paxos): (base/cluster_config.c:281) SuccessionList[2]: Node bb900c900000065 : Port 3001 ; GroupID 201 ; NodeID 101 [Self]
```

### Where to Next?
- Configure [service, fabric, and info sub-stanzas](/docs/operations/configure/network/general) which defines
  what interface will be used for application to node communication.
- Configure [heartbeat sub-stanza](/docs/operations/configure/network/heartbeat) which defines what interface
  will be used for intracluster communications.
- Learn more about [Rack Aware Architecture](/docs/architecture/rack-aware.html).
- Or return to [Configure Page](/docs/operations/configure).
