---
title: Network Heartbeat Configuration
description: Ensure cluster integrity with Aerospike's heartbeat protocols so your service. 
---

Aerospike's heartbeat protocols are responsible for maintaining cluster
integrity. There are two supported heartbeat modes:
* Multicast (UDP)
* Mesh (TCP)

{{md (path-resolve (path-dirname page.src) "cloud_consideration.part.md") }}
{{md (path-resolve (path-dirname page.src) "multicast.part.md") }}
{{md (path-resolve (path-dirname page.src) "mesh.part.md") }}


### Where to Next?
- Configure [service, fabric, and info sub-stanzas](/docs/operations/configure/network/general) which defines
  what interface will be used for application to node communication.
- (Optional) Configure [Rack Aware](/docs/operations/configure/network/rack-aware) which enables Aerospike to support
  top-of-rack switch failure.
- Learn more about the [Clustering Architecture](/docs/architecture/clustering.html).
- Or return to [Configure Page](/docs/operations/configure).
