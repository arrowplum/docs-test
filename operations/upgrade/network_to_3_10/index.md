---
title: Network Configuration Migration Guide for 3.10 Release
description: Understanding network and heartbeat changes in 3.10.
styles:
  - /assets/styles/ui/steps.css
---

In server version 3.10, Aerospike has revamped it's network and heartbeat capabilities along with some updates to the configuration. This document covers the upgrade path for your existing clusters, and points out the major features that these changes bring. The different network sub-stanza's are - *Service*, *Heartbeat*, *Fabric*, and *Info*. Some configurations are deprecated or renamed, and some require adding new configuration, and some take on additional meaning. **Please make sure you read through ALL sections to ensure ALL required changes are made.**

Details on each configuration can also be found on the [Configuration Reference Manual](/docs/reference/configuration).

{{#note}}
Versions 3.10 and pre-3.10 are **cluster-compatible**, so a rolling restart upgrade with mixed versions is supported. Please keep a copy of your original aerospike.conf configuration file, in case of an unlikely event requiring a downgrade. When upgrading to the initial version of 3.10 (3.10.0.3), and using the asadm version that came bundled with it (asadm version 0.1.5) using a 3.10.0.3 upgraded node as the 'seed' node for asadm will only display upgraded nodes. Pointing asadm to a node running a previous version will discover the whole cluster.
{{/note}}


{{#warn}}
If you have automated scripts that restart Aerospike server, please include an additional step to verify the interfaces are up before starting up Aerospike server.
{{/warn}}

## Network:Heartbeat
This sub-stanza defines network configurations related to sending and receiving heartbeats node-to-node.

If your heartbeat mode is multicast, please follow the procedure in "Multicast" section. If your heartbeat mode is mesh, please following the procecure in "Mesh" section. 

### Multicast

1. Rename `address` to `multicast-group`. `multicast-group` continues to be the group identifier for the set of nodes which should form a cluster by exchanging multicast datagrams. This configuration is a mandatory configuration. There can be multiple instances.

2. `interface-address` is obsolete. If you define an `interface-address`, please change it to `address`. Please also see Fabric section below on additional changes required.

3. `address` is now defined as the bind address, and is optional. This configuration allows Aerospike daemon to accept connections only on the specified address. There can be zero or more `address` configuration directives. When unspecified, Aerospike daemon will listen for heartbeat broadcasts on all interfaces (`address any`). 

4. If you have `mcast-ttl` parameter, please rename it to `multicast-ttl`.

5. `reuse-address` is obsolete. Please remove it.

Example configuration of your current pre-3.10 version multicast cluster `heartbeat` stanza:

```
  heartbeat {
    mode multicast                  # Send heartbeats using Multicast
    address 239.1.99.2              # multicast address
    port 9918                       # multicast port
    interface-address 192.168.1.100 # (Optional) (Default any) IP of the NIC to
                                    # use to send out heartbeat and bind
                                    # fabric ports
    interval 150                    # Number of milliseconds between heartbeats
    timeout 10                      # Number of heartbeat intervals to wait
                                    # before timing out a node
  }
```

Updated configuration in 3.10+ server versions:

```
  heartbeat {
    mode multicast                  # Send heartbeats using Multicast
    multicast-group 239.1.99.2      # multicast address
    port 9918                       # multicast port
    address 192.168.1.100 # (Optional) (Default any) IP of the NIC to
                                    # use to send out heartbeat and bind
                                    # fabric ports
    interval 150                    # Number of milliseconds between heartbeats
    timeout 10                      # Number of heartbeat intervals to wait
                                    # before timing out a node
  }
```


### Mesh

1. `interface-address` is obsolete. If you define an `interface-address`, please change it to `address`. Please also see Fabric section below on additional changes required.

2. `address` parameter continues to be the bind address (ie, the address that Aerospike daemon accepts incoming conections on). In addition, there can now be zero or more such directives, and the value can be a valid network interface name such as `eth0`. If not specified, or specified as `any`, the daemon will accept mesh heartbeat connections on all interfaces. `address` also dictates the ip addresses announced to peer-nodes to connect back.

3. `reuse-address` is obsolete. Please remove it.

Example configuration of your current pre-3.10 version mesh cluster `heartbeat` stanza:
```
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
```

Updated configuration in 3.10+ server versions:

```
  heartbeat {
    mode mesh                   # Send heartbeats using Mesh (Unicast) protocol
    address 192.168.1.100       # (Optional) (Default: any) IP's of the NIC on
    address 192.168.100.132     # which this node is listening to heartbeat
    port 3002                   # Port on which this node is listening to
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
```

## Network:Fabric
This sub-stanza defines the node-to-node fabric traffic configuration.

1. Starting server version 3.10+, we can now have one or more `address` configuration directives to specify the bind address for accepting incoming fabric connections. In previous versions, the `address` directive was ignored.

2. If you previously have `interface-address` specified in your network.heartbeat section (both multicast and mesh), you MUST also specify the same value as `address` here in the fabric section, to ensure node-to-node network traffic isolation.

3. If you previously have `network-interface-name` specified in your network.service section, you MUST also specify the same IP address dictated by the interface, as the `address` in the fabric section, to ensure node-to-node network traffic isolation.

4. In addition, in the rare case that you may desire to isolate fabric network traffic from heartbeat traffic, it can be done after switching your heartbeat protocol version to v3, which will also announce the `address` value to peer-nodes, to be used for other node connecting back. 

In pre-3.10 server versions:

```
  fabric {
    address any
    port 3001   # Intra-cluster communication port (migrates, replication, etc).
  }
```

In 3.10+ server versions:

```
  fabric {
    address 192.168.100.125
    port 3001   # Intra-cluster communication port (migrates, replication, etc).
  }
```

## Network:Service
This sub-stanza defines the client traffic access configuration.

1. The basic client service configuration remain as is. However, starting server 3.10+, you can specify multiple `address` and `access-address`. If, for example, the section contains four `address` directives, we listen on all four addresses for incoming client connections. If it contains two `access-address` directives, then we announce to clients the IP addresses specified by both.

2. Similar to before, `access-address` can be a NAT'ed IP address, such as the public ip in AWS. `access-address` now supports both IP address and DNS names. They are not resolved but are simply syntax validated. 

3. `alternate-address` is deprecated. If you have `alternate-address` specified, replace it with [`alternate-access-address`](/docs/reference/configuration#alternate-access-address). 

4. `network-interface-name` is deprecated. Use `node-id-interface` configuration at the global service (Not the Network service) level as a replacement to have the 'Node ID' generated based on specific interface's MAC address. Refer to the Fabric section for additonal changes.

5. `reuse-address` is obsolete. Please remove it.

In 3.10+ server versions:

```
  service {
    address 192.168.1.100        # Bind address 
    address 192.168.100.125      # Bind address

    port 3000                    # port on which the service is listening.
    access-address 192.168.1.100 # First IP address announced to clients to access the service.
    alternate-access-address 192.168.100.125  # Second IP address announced to clients to access the service.
  }
```

We have defaults for any un-specified configurations in the service stanza.
- Default `address` configuration is `any`, i.e. all IP addresses.
- If no `access-address` is specified, values in `address` will be used as `access-address`.

## Network:Info
This sub-stanza defines the telnet port backdoor which can similar service Aerospike's info commands.

1. Similar to other network stanza's, starting server version 3.10+, we can now have one or more `address` configuration directives to specify the bind address for accepting incoming telnet connections. In previous versions, the `address` directive was ignored.

## Summary on configuration semantic
- `address` in all sections always mean a bind address. When unspecified, it defaults to `any`, and queries all available interfaces. Localhost addresses are always added as bind addresses.
- `address` can now accept the following as valid values - IPv4 address (e.g. 1.2.3.4), IPv6 address (e.g. 2001::1234, DNS name (e.g. cluster.aerospike.com) or a network interface name (e.g. eth0). Note DNS name or network interface names are resolved to IP addresses when the configuration file is parsed, i.e., at Aerospike daemon startup. 
- `address` is also the default announce address, unless there is an `access-address` directive to override it.
- `access-address` can now be IPv4 address, IPv6 address or DNS name.
- The concept of bind addresses and access addresses has been extended from single addresses to lists of addresses. We can now bind to multiple interfaces and announce multiple access addresses. The way of specifying multiple addresses is to simply repeat the configuration directive for a single address, e.g., to use multiple access-address directives in a configuration section.

