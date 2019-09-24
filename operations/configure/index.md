---
title: Configure Aerospike Database
description: Learn how easy it is to configure the Aerospike Database
---

Aerospike uses a single file to configure a database node which can be found at
`/etc/aerospike/aerospike.conf`. The configuration file is divided into different
*contexts*, some of which are optional. These *contexts* can be in any order within the configuration file. A context may be further divided into
*sub-contexts*. The following is an example of an empty configuration file
including most common *contexts* and *sub-contexts*.

```ruby
service {}               # Tuning parameters and process owner

network {                # Used to configure intracluster and application-node
                         # communications
    service {}           # Tools/Application communications protocol
    fabric {}            # Intracluster communications protocol
    info {}              # Administrator telnet console protocol
    heartbeat {}         # Cluster formation protocol
}

security {               # (Optional, Enterprise Edition only) to enable 
                         # ACL on the cluster
    enable-security true
}

logging {}               # Logging configuration

xdr {                    # (Optional, Enterprise Edition only) Configure Cross
                         # Datacenter Replication
    datacenter <name> {} # Remote datacenter node list
}

namespace <name> {       # Define namespace record policies and storage engine
    storage {}           # Configure persistence or lack of persistence
    set {}               # (Optional) Set specific record policies
}
```

### Configuration Steps
1. Configure [Network](/docs/operations/configure/network) service and heartbeat sub-contexts
2. Configure [Namespaces](/docs/operations/configure/namespace)
3. Configure [Logging](/docs/operations/configure/log) and [Log Rotate](/docs/operations/configure/log/index.html#log-rotate)
4. (Optional) Configure [Security](/docs/operations/configure/security)
5. (Optional) Configure [Rack-Aware](/docs/operations/configure/network/rack-aware)
6. (Optional) Configure [Cross Datacenter Replication](/docs/operations/configure/cross-datacenter)

### Where to Next
- [Troubleshooting Guide](/docs/operations/troubleshoot)
- [Knowledge Base](https://discuss.aerospike.com/c/knowledge-base)
- Reference Manuals:
  - [Configuration Parameters](/docs/reference/configuration)
  - [Info Commands](/docs/reference/info)
  - [Metrics](/docs/reference/metrics)
  - [Server Log Messages](/docs/reference/serverlogmessages)
