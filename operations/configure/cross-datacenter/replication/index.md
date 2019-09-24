---
title: XDR Replication Configuration
description: Configure rules to manage your Aerospike Cross Data Center Replication (XDR) for whitelisting and blacklisting.
---

The Aerospike configuration can define which namespaces in a cluster are to be
shipped and which location to ship that data. The namespace stanza may also
define which sets within a namespace should be shipped or not shipped.

### XDR Namespace Replication Setup
To configure a namespace for replication add the following:

- Set `enable-xdr` to `true` for the namespaces to be replicated.
- Configure one or more remote datacenters for replication with the
  `xdr-remote-datacenter` parameter.

```ruby
...
xdr {
    ...
}

namespace <namespace-name-1> {
    enable-xdr true          # Enable replication for this namespace.

    # List of datacenters to replicate data to.
    xdr-remote-datacenter <datacenter-name-1>
    xdr-remote-datacenter <datacenter-name-2>
}

namespace <namespace-name-2> {
    # Without "enable-xdr true" this namespace *will not* replicate.
    ...
}
...
```

### XDR Set Replication Setup

{{#warn}}Please Note this functionality is only available from Aerospike Version 3.0.27+. Please consider upgrading to the latest version if you are planning to use these features.{{/warn}}

{{#info}}The following configuration parameters are dynamic: [sets-enable-xdr](https://www.aerospike.com/docs/reference/configuration/#sets-enable-xdr) and [set-enable-xdr](https://www.aerospike.com/docs/reference/configuration/#set-enable-xdr).{{/info}}

XDR's set replication has two modes of operation, whitelisting and blacklisting.
With whitelisting, XDR will not replicate any set data unless there is a rule
explicitly in the configuration to do so. Blacklisting requires a rule to not
replicate, otherwise all sets will replicate to remote clusters.

#### Configuring Set Whitelisting

   To configure set whitelisting, make the following configurations:
   1. Set `enable-xdr` to `true` for the namespaces to be replicated.
   2. Configure one or more remote datacenters to replicate data to.
   3. Set `sets-enable-xdr` to `false`, this instructs XDR to require a
      namespace to be whitelisted to allow a set to replicate.
   4. For each set that needs replication, set `set-enable-xdr` to `true` in the
      `set` sub-stanza.

   ```ruby
   ...
   xdr {
       ...
   }

   namespace <namespace-name> {
       enable-xdr true      # Enable replication for this namespace.

       # List of datacenters to replicate data to.
       xdr-remote-datacenter <datacenter-name-1>
       xdr-remote-datacenter <datacenter-name-2>

       sets-enable-xdr false # Do not replicate sets unless instructed.
       ...
       set <set-name-1> {
           set-enable-xdr true # Replicate this set (whitelisted).
           ...
       }

       set <set-name-2> {
           # without "set-enable-xdr true" this set *will not* replicate.
           ...
       }
       ...
   }
   ...
   ```
   <br>

#### Configuring Set Blacklisting

   To configure set blacklisting, make the following configurations:
   1. Set `enable-xdr` to `true` for the namespaces to be replicated.
   2. Configure one or more remote datacenters to replicate data to.
   3. Set `sets-enable-xdr` to `true`, this instructs XDR to require a
      namespace to be blacklisted to prevent the set from replicating.
   4. To prevent a set from replicating, set `set-enable-xdr` to `false` in the
      `set` sub-stanza

   ```ruby

   ...
   xdr {
       ...
   }

   namespace <namespace-name> {
       enable-xdr true      # Enable replication for this namespace
       # List of datacenters to replicate data to.
       xdr-remote-datacenter <datacenter-name-1>
       xdr-remote-datacenter <datacenter-name-2>

       sets-enable-xdr true # Replicate all sets unless instructed not to.
       ...
       set <set-name-1> {
           # without "set-enable-xdr false" this set *will* replicate
           ...
       }

       set <set-name-2> {
           set-enable-xdr false # Do not replicate this set (blacklisted).
           ...
       }
       ...
   }
   ...
   ```

### Where to Next?
- Configure [XDR Inter-cluster Networking](/docs/operations/configure/network) which defines other
  datacenters and if applicable the private to public ip address translation.
- Learn about [XDR Architecture](/docs/architecture/xdr.html).
- Return to [Datacenter Replication](/docs/operations/configure/cross-datacenter).
