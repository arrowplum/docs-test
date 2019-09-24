### Mesh (Unicast) Heartbeat
Mesh uses TCP point to point connections for heartbeats. Each node in the cluster
maintains a heartbeat connection to all other nodes, resulting in many
connections required for mesh. For this reason, we recommend using multicast
heartbeat protocol when available.

{{#note}}
See [Upgrade-Network to 3.10](/docs/operations/upgrade/network_to_3_10#mesh) for more details if upgrading from versions prior to 3.10.
{{/note}}

#### Configuration Steps
In the heartbeat sub-stanza:

1. Set `mode` to `mesh`.
2. (Optional) Set `address` to the IP of the local interface intended for intracluster
   communication. This setting also controls the interface **fabric** will use.
   Needed when isolating intra-cluster traffic to a particular network interface.
3. Set [`mesh-seed-address-port`](/docs/reference/configuration/#mesh-seed-address-port) to be the IP address (or qualified DNS name as of version 3.10) and heartbeat port of a node in the cluster.
4. Set [`interval`](/docs/reference/configuration/#interval) and [`timeout`](/docs/reference/configuration/#timeout)
   * [`interval`](/docs/reference/configuration/#interval) (recommended: 150) controls how often to send a heartbeat
     packet.
   * [`timeout`](/docs/reference/configuration/#timeout) (recommended: 10) controls the number of intervals after which a node is considered to be missing by the rest of the nodes in the cluster if they haven't received the heartbeat from the missing node.
   * With the recommended settings, a node will be aware of another node leaving the
     cluster within 1.5 seconds.

{{#warn}}
When using fully qualified names in versions 4.3.1 and earlier, names that would not DNS resolve could cause 
clusters to split if the DNS server slows down and the name resolution takes longer to fail. A successful 
DNS resolution will replace the name with the IP address until the subsquent restart. 
{{/warn}}

#### Example

```
...
  heartbeat {
    mode mesh                   # Send heartbeats using Mesh (Unicast) protocol
    address 192.168.1.100       # (Optional) (Default: any) IP of the NIC on
                                # which this node is listening to heartbeat
    port 3002                   # port on which this node is listening to
                                # heartbeat
    mesh-seed-address-port 192.168.1.100 3002 # IP address for seed node in the cluster
                                              # This IP happens to be the local node
    mesh-seed-address-port 192.168.1.101 3002 # IP address for seed node in the cluster
    mesh-seed-address-port 192.168.1.102 3002 # IP address for seed node in the cluster
    mesh-seed-address-port 192.168.1.103 3002 # IP address for seed node in the cluster
    
    interval 150                # Number of milliseconds between heartbeats
    timeout 10                  # Number of heartbeat intervals to wait before
                                # timing out a node
  }
...

```
