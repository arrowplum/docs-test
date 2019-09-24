---
title: Network Configuration
description: Aerospike Databases' Network configuration stanza sets up critical network ports to be used by other nodes, application, and tools.
---

Aerospike Databases' Network configuration stanza sets up critical network
ports to be used by other nodes, application, and tools. The following table lists and describes ports used by Aerospike Database and XDR:

| Name                    | Default Port | Description                                                                                                                                                        |
|-------------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Service                 | 3000         | Application, Tools, and Remote XDR use the Service port for database operations and cluster state.                                                                        |
| Fabric                  | 3001         | Intra-cluster communication port. Replica writes, migrations, and other node-to-node communications use the Fabric port.                                                          |
| Mesh Heartbeat          | 3002         | Heartbeat protocol ports are used to form and maintain the cluster. (Only one heartbeat port may be configured.)                                                    |
| Multicast Heartbeat     | 9918         | Heartbeat protocol ports are used to form and maintain the cluster. (Only one heartbeat port may be configured.)                                                    |
| Info                    | 3003         | Telnet port that implements a plain text protocol for administrators to issue info commands. For more information, see [asinfo documentation](/docs/tools/asinfo). |                                                                            |

{{#note}}
Ensure that all Application and XDR nodes can communicate to the
**service** port on all Aerospike nodes. Also ensure that each node can
communicate over the configured **heartbeat** and **fabric** ports. 
{{/note}}

{{#note}}
For versions prior to 3.8, port 3004 is the default port to query the health state of the Cross Datacenter Replication (XDR) client.
{{/note}}

# Where to Next?
- Configure [service, fabric, and info sub-stanzas](/docs/operations/configure/network/general) which defines
  what interface will be used for application to node communication.
- Configure [heartbeat sub-stanza](/docs/operations/configure/network/heartbeat) which defines what interface
  will be used for intracluster communications.
- (optional) Configure [Rack Aware](/docs/operations/configure/network/rack-aware) which enables Aerospike to support
  top-of-rack switch failure.
- Or return to [Configure Page](/docs/operations/configure).
