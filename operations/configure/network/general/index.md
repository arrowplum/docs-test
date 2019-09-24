---
title: General Network Configuration
description: |
  Learn the general network configurations to get the Aerospike up and running quickly.
---

The network stanza of the Aerospike configuration requires the following
sub-stanzas:
* service
* heartbeat
* fabric
* info

Typically, only the service and heartbeat sub-stanzas need modification depending on if you require to isolate the fabric (inter-node replication,migration) traffic from heartbeat and service.

{{#note}}
See [Upgrade-Network to 3.10](/docs/operations/upgrade/network_to_3_10) for more details if upgrading from versions prior to 3.10.
{{/note}}

Here is a general definition for the configurations in the 'service' sub-stanza1:

- the `address` configuration refers to interfaces or IP addresses to bind and listen to. For version 3.10 and above, you can configure multipe IP addresses to bind to.
- the `access-address` configuration refers to interfaces or IP addresses to publish for clients (typically clients within the same subnet/DC).
- the `alternate-access-address` configuration refers to interfaces or IP addresses to publish for clients who wouldn't be able to connect to the access-address interfaces or IP addresses. If the interface / IP specified here are actual interfaces (rather than mapped over NAT), then the corresponding `address` config has to be specified as well (unless if `address any` is set).

## For server version 3.10 and above:

**Service**

Example use cases:

1- Host with 2 network interfaces, x.x.x.x and y.y.y.y, with x.x.x.x for clients within the same subnet/DC (private IP) and y.y.y.y for clients in a different subnet/DC (public IP). The IP address y.y.y.y is not mapped over NAT:

```
service {
    address x.x.x.x
    address y.y.y.y
    access-address x.x.x.x
    alternate-access-address y.y.y.y
}
```

The `access-address` x.x.x.x is to prevent the y.y.y.y IP to also be broadcasted (if `access-address` is not specified, all IPs specified as `address` are published/broadcasted).

2- If the y.y.y.y IP is mapped over NAT:

```
service {
    address x.x.x.x
    access-address x.x.x.x
    alternate-access-address y.y.y.y
}
```

Or, as the `address` would be published by default (unless if overwritten through access-address):

```
service {
    address x.x.x.x
    alternate-access-address y.y.y.y
}
```

3- Alternate configuration that works in most cases: `address any` (will bind to all available interfaces) and then publishing the specific `access-address` and `alternate-access-address`.

```
service {
    address any
    access-address x.x.x.x
    alternate-access-address y.y.y.y
}
```

** Info and Fabric **

The intra-cluster communication traffic can be isolated from the regular client traffic by specifying the `address` configuration in the fabric and info respective sub-stanzas. By default, they are set to `any`.

```
fabric {
  address any
  port 3001   # Intra-cluster communication port (migrates, replication, etc).
}

info {
  address any
  port 3003   # Plain text telnet management port.
}
```

## For server versions prior to 3.10:

In the service sub-stanza, ensure that `access-address` is configured to the network
interface that the Application will communicate to. For server versions 3.3.26 to 3.9.1.1, this `access-address` should 
be one of the physical network interface address. In case of vitual access-address
use the `virtual` keyword in the end, e.g. `access-address 192.168.1.100 virtual`.

```bash
...
network {
  service {
    address any                  # IP of the NIC on which the service is
                               # listening.
    port 3000                    # port on which the service is listening.
    access-address 192.168.1.100 # IP address exported to clients that access
                               # the service.
  }

  fabric {
    address any
    port 3001   # Intra-cluster communication port (migrates, replication, etc).
  }

  info {
    address any
    port 3003   # Plain text telnet management port.
  }

  heartbeat {
    mode multicast                  # Send heartbeats using Multicast
    address 239.1.99.2              # multicast address
    port 9918                       # multicast port
    interface-address 192.168.1.100 # IP of the NIC to use to send out heartbeat
                                    # and bind fabric ports
    interval 150                    # Number of milliseconds between heartbeats
    timeout 10                      # Number of heartbeat intervals to wait
                                    # before timing out a node
  }

#  heartbeat {
#    mode mesh                   # Send heartbeats using Mesh (Unicast) protocol
#    address 192.168.1.100       # IP of the NIC on which this node is listening
#                                # to heartbeat
#    port 3002                   # port on which this node is listening to
#                                # heartbeat
#    mesh-seed-address-port 192.168.1.101 3002 # IP address for seed node in the cluster
#    mesh-seed-address-port 192.168.1.102 3002 # IP address for seed node in the cluster
#    interval 150                # Number of milliseconds between heartbeats
#    timeout 20                  # Number of heartbeat intervals to wait before
#                                # timing out a node
#  }
}
...
```
### What is access-address?

The `access-address` parameter determines the address which is published for
client access. It is recommended when an Aerospike node has multiple IP
addresses to publish an `access-address` pointing to desired IP in the config.

#### When to use access-address?

An `access-address` should be specified when you want the clients to communicate
on only one of the IPs' but want the server to listen on multiple IPs. This
might be required when setting up XDR destination. The `access-address` should
be specified to the IP which the clients should be using. Typically, when a
server has a public and a private IP, the private IP is used for clients usage
whereas public IP is used for XDR purposes. 

If the private access-address of the
destination cluster is not reachable from source cluster of XDR and the
destination has additional public address on which aerospike is listening (and
these public address are reachable from source cluster), then it is important
to specify one of the following:

- [`dc-int-ext-map`](/docs/operations/configure/cross-datacenter/network/index.html#general-xdr-network-configuration)
on the source cluster's config for mapping private IP to public ondes.
- [`alternate-access-address`](/docs/reference/configuration#alternate-access-address) on the destination cluster along with [`dc-use-alternate-services`](/docs/reference/configuration#dc-use-alternate-services) on the source cluster.

#### What happens when access-address is missing?

If the nodes have multiple IP addresses, the clients will see multiple server
IPs for each node. Some of the clients (like java) can de-duplicate the
duplicate IPs based on node-id. Where as some tools
([AMC](/docs/amc), [asadm](/docs/tools/asadm)) may not be able
to de-duplicate. Tools like asadm may report such scenarios with cluster
visibility false error since it sees a mismatch between cluster size and number
of server IPs.

This does not impact the working of clients but only causes a problem with the
tools.

### Next Steps
- Configure [heartbeat sub-stanza](/docs/operations/configure/network/heartbeat) which defines what interface
  will be used for intra-cluster communications.
- Configure [Rack Aware](/docs/operations/configure/network/rack-aware) which enables Aerospike to support
  top-of-rack switch failure.
- Or return to [Configure Page](/docs/operations/configure).
