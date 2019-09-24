---
title: Network
description: Learn best practices on how Aerospike uses the network.
scripts:
  - /assets/scripts/utils/target_blank.js
---

##Aerospike Network Usage##

- All connections are made through standard TCP, transporting Aerospike developed wire-protocol.
- Heartbeat signals can be sent in multicast mode, or mesh mode (node-to-node via TCP).
- Typical ports that are open -
 - 3000 - 'service' port, which handles client requests and responses.
 - 3001 - 'fabric' port, which handles server node-to-node communication.
 - 3002 - 'heartbeat' port - (optional) used for heartbeat TCP messages.
 - 3003 - 'info' port, which is a telnet port which can be used for Aerospike info commands.
 - 3004 - 'xdr' port, used by legacy xdr daemon to communicate with main Aerospike daemon.
- Aerospike does not make use of specialized network equipment or drivers.
- 1Gb and 10Gb are both supported.

##Network Topology Requirement##
- To ensure the integrity and protection of data, Aerospike client and server nodes MUST be deployment within a secure network environment such as a VPN. Unknown clients must NOT be allowed to connect to the aerospike TCP ports.
- To ensure performance, minimize intermediate network devices such as routers or multiple switches between client node and server nodes.
- Server nodes are recommended to be physically as close as possible with minimal ping time, so a tightly coupled stable cluster can be maintained.
- Refer to the [TLS Configuration](/docs/operations/configure/network/tls) page for details on setting up TLS.


##Other Recommendations##

- One important thing to realize is that it is difficult to get above 100K TPS per node with default settings. This limit is often more than enough for most use cases. However, in the event you wish to get more there are a few ways to do this:
  - In most cases, the limiting factor is not the actual bandwidth, but the system interrupts. By using the “top” command and pressing “1″, you can see how the CPU cores are spending their time. If the “si” column is high on any core, this may be due to sharing of a single interrupt on the card. If you have multiple interrupts available, you can spread the interrupts to different cores. In particular, we have found that network interfaces using Intel chipsets. Check with the manufacturers of your network cards to see if they allow for multiple interrupts. Click [here](https://cs.uwaterloo.ca/~brecht/servers/apic/SMP-affinity.txt) to see how to balance interrupts across cores. 
  - You may be able to use link aggregation to make use of more than ethernet port. This will bond multiple ports together to increase throughput. A good description of the protocol for this (802.3ad) can be found at: http://en.wikipedia.org/wiki/Link_aggregation. Note that the exact mechanism to do this varies depending on your hardware. Aerospike customers have used to this to aggregate 4-1 Gb ports to act as a single 4 Gb connection. This acts below the TCP level, so no changes are necessary for the Aerospike configuration.
  - You could choose to use 10 Gb networking equipment. While not generally a requirement, this is sometimes preferable to using link aggregation.
- Some customers have asked if they can use more than one Ethernet port. In this case one would face the web/application servers (service traffic) and the other would be used for communication with other nodes and for administration (intra-cluster). While not a requirement, Aerospike was intended to operate in this network architecture. In the event that a node is lost, data will be transferred from one server to another. By separating out the service traffic from the intra-cluster traffic, you can maintain greater reliability on the service traffic. Check the instructions for multiple network connectors.
- In some cases customers have used load balancers between the  web/application servers and the Aerospike cluster. Because Aerospike already handles the load balancing, this can result in some confusion. It is highly recommended to turn off any third-party load balancer.
- There are sometimes controls in place to detect denial-of-service attacks, such as Storm Control. Because the way that applications access Aerospike, these may look like attacks, resulting in a degradation or shutdown of service. It is highly recommended to turn off Storm Control.

&nbsp;

---

<div class="clearfix"></div>
<div class="pull-left">
<a class="btn btn-default" href="/docs/operations/plan/ssd">&lsaquo; Flash Storage</a>
</div>
<div class="pull-right">
<a class="btn btn-primary" href="/docs/operations/plan">Done &rsaquo;</a>
</div>
<div class="clearfix"></div>

&nbsp;
