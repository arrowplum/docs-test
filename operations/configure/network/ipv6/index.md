---
title: IPv6 Configuration
description: Configure cluster to use IPv6. 
---

Since 3.10 release, Aerospike clusters can be configured to use IPv6 for both node-to-node and client-to-cluster connections.

This document describes prerequisites and relevant configurations. If you are planning to migrate your network to an IPv6-only environment, please contact Aerospike Support for a smooth migration.

## Required Network Setup for IPv6:
1. IPv6 enabled Linux OS.
2. Site-local or global IPv6 address on an interface. Aerospike IPv6 does not support link-local addresses.

Sample configuration on a IPv6 enabled interface:

```
[citrusleaf@localhost serverlocal]$ ifconfig
eth1      Link encap:Ethernet  HWaddr 00:0C:29:A9:DF:10 
          inet addr:172.16.1.128  Bcast:172.16.1.255  Mask:255.255.255.0
          inet6 addr: 2001::1111/64 Scope:Global
          inet6 addr: fe80::20c:29ff:fea9:df10/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:387 errors:0 dropped:0 overruns:0 frame:0
          TX packets:324 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:34012 (33.2 KiB)  TX bytes:31786 (31.0 KiB)
```

For the above, address "2001::1111" will work, because it's global. address "fe80::20c:29ff:fea9:df10" will not, because it's link-local.
If your network is not IPv6-enabled and thus doesn't assign you an IPv6 address, you might still be able to work with IPv6 by manually assigning IPv6 addresses to machines. For this, the machines (clients as well as cluster nodes) need to reside in the same Ethernet segment.

## Required Client:
- Java - version 3.3 and above.
- C - version 4.10 and above.
- C# - version 3.3 and above.


## Additional Requirement -
- Available on Enterprise builds only.
- `advertise-ipv6` must be true. 
- Must be on heartbeat protocol version v3. **Please contact Aerospike support on upgrading to heartbeat v3 protocol**.


## Sample Aerospike configuration:

```
service {
    ....
    advertise-ipv6 true
}
 
network {
    heartbeat {
        ....
        protocol v3 // MUST follow instruction on how to switch over to hbv3 protocol.
    }
}
```
