### Multicast Heartbeat
We recommend using the multicast heartbeat protocol when available. For various
reasons your network may not support multicast. See our
[troubleshooting guide](/docs/operations/troubleshoot) for information on
how to validate multicast in your environment.

{{#note}}
See [Upgrade-Network to 3.10](/docs/operations/upgrade/network_to_3_10#multicast) for more details if upgrading from versions prior to 3.10.
{{/note}}

#### Configuration Steps
In the heartbeat sub-stanza:

1. Set `mode` to `multicast`.
2. Set `multicast-group` to a valid multicast address (239.0.0.0-239.255.255.255).
3. (Optional) Set `address` to the IP of the interface intended for intracluster
   communication. This setting also controls the interface **fabric** will use.
   Needed when isolating intra-cluster traffic to a particular network interface.
4. Set `interval` and `timeout`
    * `interval` (recommended: 150) controls how often to send a heartbeat
     packet.
    * `timeout` (recommended: 10) controls the number of intervals after which a node is considered to be missing by rest of nodes in the cluster if they haven't received the heartbeat from missing node.
    *  With the default settings, a node will be aware of another node leaving the
     cluster within 1.5 seconds.

#### Example

```bash
...
  heartbeat {
    mode multicast                  # Send heartbeats using Multicast
    multicast-group 239.1.99.2              # multicast address
    port 9918                       # multicast port
    address 192.168.1.100 # (Optional) (Default any) IP of the NIC to
                                    # use to send out heartbeat and bind
                                    # fabric ports
    interval 150                    # Number of milliseconds between heartbeats
    timeout 10                      # Number of heartbeat intervals to wait
                                    # before timing out a node
  }
...
```

