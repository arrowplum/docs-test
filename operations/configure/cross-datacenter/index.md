---
title: Cross Datacenter Replication (XDR)
description: The Cross Datacenter Replication (XDR) service is bundled with Aerospike Enterprise Edition and provides inter-cluster replication at the namespace or optionally set granularity level.
---

The Cross Datacenter Replication (XDR) service is bundled with Aerospike
Enterprise Edition and provides inter-cluster replication at the namespace or
optionally set granularity level. As of Aerospike Server Enterprise Edition 
version 3.8, the Cross Datacenter Replication (XDR) service is built in within 
the main Aerospike service (asd).

XDR adds a new stanza, "xdr", to the configuration. This stanza defines if XDR is 
enabled, paths to XDR specific files, and remote datacenter configurations.

Additionally, XDR requires the replicated namespaces to explicitly enable
XDR, and to indicate the remote datacenter for replication.

{{#warn}}
For Aerospike Server versions prior to 4.7 XDR does not support LDAP logins.
{{/warn}}

## Versions 3.8 and above

### XDR stanza basic configuration
The basic configuration consists of enabling XDR and configuring XDR log locations. Here is an example of a basic xdr stanza configured:

  ```
  xdr {
        enable-xdr true 
        xdr-digestlog-path /opt/aerospike/xdr/digestlog 100G 
 
        datacenter DC1 {
                dc-node-address-port xx.xx.xx.xx 3000
                dc-node-address-port yy.yy.yy.yy 3000
                dc-node-address-port zz.zz.zz.zz 3000
        }
  }
  ```

The namespace that should be replicated to the configured DC then needs to refer to a DC and have 
xdr enabled as well:

  ```
  namespace test {
        enable-xdr true 
        xdr-remote-datacenter DC1
        ...
  }
  ```

Here are the step by step details:

1. **Enable XDR.** To enable XDR, navigate to the `xdr` stanza and set `enable-xdr` to `true`.

2. **Digest Log.** Set the [`xdr-digestlog-path`](/docs/reference/configuration#xdr-digestlog-path) 
  to a file path where the XDR service has permission
  to write. By convention, `/opt/aerospike/digestlog/` is used. The parameter
  also requires a max size which is a number that specifies the maximum number
  of bytes the digestlog may use. Each digest logged in the digest log will use
  80 bytes. When full, the old digest are overwritten with new ones being written
  (circular buffer).

3. **Remote datacenter(s).** For XDR to be able to communicate with the remote cluster 
  the configuration must provide at least one `dc-node-address-port` entry in the 
  `datacenter` sub-stanza that describes an Aerospike address and service port for a 
  node in the remote cluster. It is recommended to provide many nodes in this 'seed' list.

  ```
  xdr {
  ...
      datacenter DC1 {
          dc-node-address-port xx.xx.xx.xx 3000
          dc-node-address-port yy.yy.yy.yy 3000
          dc-node-address-port zz.zz.zz.zz 3000
      }
  }
  ```

{{#note}}
**Note:** The order of the datacenter definitions defined in Aerospike configuration, as well as each datacenter name should be identical across all nodes in the cluster.
{{/note}}

### Advanced configuration options

1. **Alternate Address.** If the remote node's addresses broadcasted for local applications aren't accessible from the local 
  cluster then the remote's configuration files must broadcast an alternate address that is accessible
  by the remote cluster. This can be configured using the `alternate-address` for versions between 3.7.1 and 3.9.1.1 on the remote side.
  Starting 3.10.0.3 this config is deprecated and `alternate-access-address` is introduced which can be used instead.<br>
  **Remote Node Config:**

  ```
  network {
      service {
          address any
          port 3000
          access-address aa.aa.aa.aa
          alternate-address xx.xx.xx.xx
      }
  }
  ```

  If alternate-address or alternate-access-address is set depending on the aerospike version, the source cluster has to specify `dc-use-alternate-services true` 
  in order to use the alternate IP address to connect to the node (instead of services).<br>
  **Source Node Config:**

  ```
  xdr {
  ...
      datacenter DC1 {
          dc-node-address-port xx.xx.xx.xx 3000
          ...
          dc-use-alternate-services true
      }
  }
  ```

2. **Dynamic datacenter(s) configuration.** Datacenters can be seeded dynamically. A minimum skeleton must
  be specified, though, providing the datacenter name. For example, the following datacenter can be seeded
  dynamically:

  ```
  xdr {
  ...
      datacenter DC2 {
      }
  }
 
  $ asinfo -v "set-config:context=xdr;dc=DC2;dc-node-address-port=xx.xx.xx.xx:3000;action=add"
  ok
  ```

  If the cluster that is dynamically configured has security enabled, data-admin privileges are required
  to dynamically change the datacenter `dc-node-address-port`.

  **It is recommended to configure datacenters in the configuration file and restart asd, to make sure the 
  configuration is correct upon a subsequent restart of a node.**

3. **Security.** If the cluster being shipped to has security enabled, add the `dc-security-config-file`
  parameter to point to a file specifying the user/password required to write in the cluster. This user
  must have write or read-write privileges. Make sure this file is adequately secured.

  ```
  xdr {
      datacenter dDC1 {
          dc-node-address-port xx.xx.xx.xx 3000
          ...
          dc-security-config-file /private/security-credentials_DC1.txt
      }
  }

  $ more /private/security-credentials_DC1.txt
  credentials
  {
      username xdr_user
      password xdr_pass
  }
  ```

  As of version 3.8.3, this file can be dynamically configured: 
  
  ```
  asinfo -v "set-config:context=xdr;dc=DC1;dc-security-config-file=/private/aerospike/security_credentials_DC1.txt"
  ```

4. Here is an example of an xdr stanza configured with a bit more options:

  ```
  xdr {
        enable-xdr true 

        xdr-digestlog-path /opt/aerospike/xdr/digestlog 100G 
        # make sure the /opt/aerospike/xdr folder exists
 
        datacenter DC1 {
                dc-node-address-port xx.xx.xx.xx 3000
                dc-node-address-port yy.yy.yy.yy 3000
                dc-node-address-port zz.zz.zz.zz 3000

                dc-use-alternate-services true
                dc-security-config-file /private/aerospike/security_credentials_DC1.txt
        }

        datacenter DC2 {
                # Can be seeded dynamically:
                # asinfo -v "set-config:context=xdr;dc=DC2;dc-node-address-port=aa.aa.aa.aa:3000;action=add" 
                # [-Udata-admin -Pdata-admin]

                # dc-use-alternate-services true
                # dc-security-config-file /private/aerospike/security_credentials_DC2.txt
        }
  }
  ```


## Versions up to 3.7.5

### Basic Configuration
The basic configuration includes enabling XDR and configuring XDR log locations.

1. To enable XDR, navigate to the `xdr` stanza and set `enable-xdr` to `true`.
2. Set the `xdr-namedpipe-path` to a file path where the XDR service has permission
   to write. By convention, `/tmp/xdr_pipe` is used.
3. Set the `xdr-digestlog-path` to a file path where the XDR service has permission
   to write. By convention, `/opt/aerospike/digestlog/` is used. The parameter
   also requires a max size which is a number that specifies the maximum number
   of bytes the digestlog may use.
4. Set the `xdr-errorlog-path` to a file path where the XDR service has permission
   to write. By convention, `/var/log/aerospike/asxdr.log` is used.
5. By convention, `xdr-pidfile` should be set to `/var/run/aerospike/asxdr.pid`.
5. Configure the `local-node-port` to the service port of the local node. The default value is `3000`.
6. Configure the `xdr-info-port` to a port XDR may listen for info protocol
   requests. The default value is `3004`.
7. (Optional) If you want the local Aerospike Server to only accept writes when
   XDR is running, set `stop-writes-noxdr true` in the **xdr** context.
   This instructs the server to reject writes originating from the application
   while XDR is down.

  ```ruby
  ...
  xdr {
      enable-xdr      true               # Used to globally enable/disable XDR on
                                         # local node.
      xdr-namedpipe-path  /tmp/xdr_pipe  # XDR to/from Aerospike communications
                                         # channel.
    
      # The digestlog is where XDR keeps track of the digests that need to be shipped.
      xdr-digestlog-path  /opt/aerospike/digestlog 100G
      # Log XDR errors
      xdr-errorlog-path   /var/log/aerospike/asxdr.log 
      # XDR PID file location                                 
      xdr-pidfile         /var/run/aerospike/asxdr.pid 
                                       
      local-node-port 3000               # Port on the local node to be used when
                                         # reading records and other operations.
      xdr-info-port   3004               # Port used by various tools to determine
                                         # health and running config status of XDR
      # stop-writes-noxdr true           # If set to true, Aerospike Server will reject
                                         # writes if XDR is down.
      ...
  }
  ...
  ```

8. For XDR to be able to communicate with the remote cluster the configuration must
  provide at least one `dc-node-address-port` entry in the `datacenter` sub-stanza
  that describes a Aerospike address and service port for a node in the remote
  cluster. If the remote node's addresses aren't accessible from the local cluster
  then the local configuration files must have a complete mapping of remote
  internal to external IP addresses using the `dc-int-ext-ipmap` parameter in the
  `datacenter` sub-stanza.

  Optionally you may define an 'xdr-compression-threshold` which may be beneficial
  if the records written by the Application layer are not already compressed and
  are typically larger than 1 KB.

  - The following examples demonstrate configuring XDR's inter-cluster networking. 
  Configure remote cluster who's IP addresses are locally accessible and use
   compression.
   ```ruby
   ...
    xdr {
        ...
        xdr-compression-threshold 1000     # (optional) Number of bytes a records
                                           # must exceed before XDR compressed the
                                           # record for shipping.

        datacenter <datacenter-name> {     # Canonical name of the remote datacenter
            # dc-node-address-port defines the remote node's IP or DNS name and the
            # service port of Aerospike on that node. Only one node in a given
            # cluster is required, XDR will discover the rest from it.
            dc-node-address-port <remote_node_ip_1> <remote-nodes-service-port>
            dc-node-address-port 172.68.17.123 3000 # example with parameters
                                                    # populated.
            ...
        }

        datacenter ...                     # Another datacenter definition
        ...
    }
    ...
    ```

  - Configure remote cluster who's IP address are not locally accessible.
   ```ruby
   ...
   xdr {
       ...
       datacenter <datacenter-name> {     # Canonical name of the remote datacenter

           # Sometimes the remote cluster may have multiple network interfaces
           # and the interface advertised, "access-address", by the remote cluster
           # isn't accessible by the local cluster. In this scenario you must
           # define all nodes in the remote cluster and a internal to external
           # mapping parameter called "dc-int-ext-ipmap. When adding a new node to
           # the remote cluster it will be necessary to update the local cluster's
           # mapping.
           dc-node-address-port <remote-node-ip-1> <remote-nodes-service-port>
           dc-node-address-port 172.68.17.123 3000 # example with parameters
                                                   # populated.

           # Node internal to external ip map, must include all remote nodes.
           dc-int-ext-ipmap <remote-node-private-ip-1> <remote-node-public-ip-1>
           dc-int-ext-ipmap 172.68.17.123 74.125.239.52 # example ip mapping with
                                                        # parameters populated.
           ...
       }

       datacenter ...                     # Another datacenter definition
       ...
   }
   ...
   ```

{{#todo markdown=true}}
  * [MANAGE] Describe using init script for xdr
  * [MONITOR] XDR monitoring, XDR stats reference (roll into metrics?)
  * [TROUBLESHOOTING] XDR common issues
  * [TUNING] threads, single-get, read/ship batch, etc
  * [CONFIGURE] Fit the following:
    * TOPOLOGIES:
      * forward-xdr-writes (very important) 
      * xdr-forward-with-gencheck (needed in ring topology)
    * Advanced?:
      * stop-writes-noxdr (Service considered down if XDR is down)
      * xdr-delete-shipping-enabled (backup cluster)
{{/todo}}

### Where to Next?
- Configure [XDR Inter-cluster Replication](/docs/operations/configure/cross-datacenter/replication) which defines which
  namespaces and optionally which sets to replicate/not replicate.
- Learn about [XDR Architecture](/docs/architecture/xdr.html).
- Learn [XDR Management](/docs/operations/manage/xdr).
- Return to [Configure Page](/docs/operations/configure).
