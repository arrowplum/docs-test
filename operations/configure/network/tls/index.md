---
title: TLS Configuration
description: Configure cluster to use TLS. 
---

Aerospike Enterprise Edition supports Transport Layer Security (TLS) for the following network traffic:

- Client-to-Cluster - Traffic between clients and all Database Nodes (as of version 3.11).
- Cluster-to-Cluster - Traffic between clusters (cross data-center replication through XDR) (as of version 3.11).
- Node-to-Node - Traffic between nodes in a cluster, both fabric and heartbeat (as of version 3.15).


This document describes options and relevant configurations. If you are planning to migrate your network to a TLS environment, contact Aerospike Support for a smooth migration.

First review the [TLS Guide](/docs/guide/security/tls.html) for an overview of the functionality. 

Then decide:
- If TLS should be run only between your client and server network, or if you have XDR and DC-to-DC traffic should also be TLS.
- If you require Standard Authentication or Mutual Authentication in each network.
- If TLS should be run between cluster nodes for fabric and/or heartbeat traffic.
- What type certificate setup you choose - Cluster-name Match, Wildcard Hostname Match, Match with Common Server Certificate, or Hostname Match with Individual Server Certificate.

Once the TLS deployment artichecture is determined, generate the appropriate certificates and follow the upgrade scenario below.

{{#warn}}
This page covers Aerospike Enterprise Edition versions 3.15 and above. Configuration syntax for TLS is different for versions 3.11 to 3.14. Refer 
to the [TLS Guide pre 3.15](/docs/guide/security/pre3_15/tls_pre3_15.html) and the [Network Configuration/TLS/Pre 3.15](/docs/operations/configure/network/tls/pre3_15) 
page for details for those specific versions.
{{/warn}}

## Configuration Parameters Reference

Below lists the relevant configuration parameters for TLS. Please also see [Configuration Reference](/docs/reference/configuration) for additional details on each parameter.

```bash
# (*)  required parameters for TLS
# (**) required parameters for standard authentication or mutual authentication
# (+)  optional parameters for mutual authentication

network {
  tls tlsname1 {                    # (*) This is the TLS name to be referred to in subsequent network 
                                    # sub-stanzas that woudld require TLS. Will be populated in the 
                                    # tls-name wire field for incoming connections. Could be set to 
                                    # <cluster-name> or <hostname> or user-defined. 
                                    # Set to `tlsname1` in this example.  
    cert-file path-to-file          # (**) Required for standard authentication or mutual authentication
    key-file path-to-file           # (**) Required for standard authentication or mutual authentication
    ca-file path-to-file            # (+) For mutual authentication or for XDR. 
                                    # Only one of ca-file / ca-path required.
    ca-path directory-path          # (+) For mutual authentication or for XDR. 
                                    # Only one of ca-file / ca-path required.
    protocols TLSv1.2               # Protocol version to include
                                    # note - In version 4.6 the default protocols configuration parameter
                                    # was changed from “-all,+TLSv1.2” to “TLSv1.2”.
    cipher-suite ALL:!COMPLEMENTOFDEFAULT:!eNULL 
                                    # Ciphers to includes. 
    cert-blacklist path-to-file     # Blacklist including serial number of certs to revoke
  }
 
  tls tlsname2 {
    ca-file /tls/ca-cert2.pem
    cert-file /tls/rem-chain2.pem
    key-file /tls/rem-key2.pem
    <...>
  }

  tls tlsname3.source.domain {
    <...>
  }

  tls <cluster-name> {
    <...>
  }

  <...>

  service {
    port 3000
    address 192.168.80.100          # Bind address for non TLS traffic. 
                                    # Default to `tls-address` if not specified.

    tls-port 4333                   # (*) Enables tls
    tls-address 192.168.90.150      # Bind address for TLS. 
                                    # Will default to `address` if not specified.
    tls-authenticate-client false|any|user-defined
                                    # If not specified, default is any, which is mutual authentication.
                                    # Multiple tls-authenticate-client entries can be defined in order 
                                    # to accept multiple subject names. At typical example would be to 
                                    # use different subject names between local clients and XDR clients 
                                    # (when specifying the subject name of course).
    tls-name <cluster-name>|<hostname>|user-defined  
                                    # (*) Which TLS config to use (from tls sub-stanza). 
                                    # This is also the name that would be populated in the tls-name 
                                    # wire field. (For example, tlsname1 to refer to the first 
                                    # tls sub-stanza described above).
  }
 
  heartbeat {
    tls-port 3012
    tls-name tlsname2
    tls-mesh-seed-address-port 192.168.90.151 3012
    <...>
  }
 
  fabric {
    tls-port 3011
    tls-name tlsname2
    <...>
  }
 
  <...>
}
 
<...>
 
xdr {
  datacenter DC1 {
    tls-node 10.10.1.1 tlsname.remote.domain 3010
                                    # Remote nodes IP address along with TLS name a cluster node expects 
                                    # the remote DC to present on XDR connections.
    <...>
    tls-name tlsname3.source.domain # The remote cluster should have a tls-authenticate-client directive
                                    # specifying this TLS name, or `false`, or `any`.
    <...>
  }

  datacenter DC2 {
    <...>                           # Can have its own set of tls configs
  }

  <...>
}
```

{{#info}}
The TLS name (specified as part of the tls sub-stanza and referred to within the service, fabric and heartbeat sub-stanzas as well as XDR stanzas) 
has the ability to be substituted. For example:<br><br>

 - [`tls`](/docs/reference/configuration#tls) / [`tls-name`](/docs/reference/configuration#tls-name) `<cluster-name>` 
 means the server will use what's defined in [`cluster-name`](/docs/reference/configuration#cluster-name) as the tls-name attribute.<br>
 - [`tls`](/docs/reference/configuration#tls) / [`tls-name`](/docs/reference/configuration#tls-name) `<hosname>` 
 means the server will use what's returned by the underlying OS's "hostname" system call as the tls-name attribute.<br><br>
{{/info}}

{{#info}}
[`ca-path`](/docs/reference/configuration#ca-path): it may be required to run a "c_rehash" to generate symlinks based on hash values.
{{/info}}

## Configuring the Certificates Blacklist

The serial number assumes hex representation, with no non-hex characters such as 'x' or ":" etc.

To add a certificate's information to the blacklist, you must have the certificate available to extract the properly formatted serial number:
```
cat chain.pem | openssl x509 -noout -serial -issuer | \
awk 'match($0,/[^=]*=(.*)/, a) { printf "%s", a[1] } END {printf "\n"}'
```

### Refreshing the Certificates Blacklist (cert-blacklist)

The certificates blacklist will automatically refresh when using Aerospike Enterprise Edition version 3.15 and above. For versions 3.11 to 3.14, the following info command can be used to refresh the list:
```
asinfo -v 'set-config:context=xdr;dc=REMOTE;tls-cert-blacklist=.../path/blacklist.txt'
```

## Upgrading to TLS

Here is a high level overview of steps for upgrades. 

### Upgrade client to cluster connections to TLS

For an existing deployment which will be upgraded to TLS, follow these steps:

- Upgrade Server
 - Rolling Upgrade each node in the cluster to a software version that is TLS aware (3.11 and above), while adding in new TLS enabled configurations. Do not shutdown clear ports.
 - Repeat for all nodes in the cluster.
 - During this time, client will continue to communicate with cluster via clear transport.
- Upgrade Client
 - Once every node is TLS enabled, start upgrading client to a software version that is TLS aware, while adding in new TLS enabled client configurations.
 - During this time, old clients will continue to communicate with cluster via clear transport, while new clients will communicate with cluster via TLS.
 - Repeat for all application nodes.
- Shut down clear (non tls) ports (optional)
 - If no clients should be allowed to reach the server, an additional Rolling Restart is required to remove and shutdown the clear port.

### Upgrade from Standard-Authentication to Mutual-Authentication

For an existing deployment already running standard-authentication to be upgraded to mutual-authentication, follow these steps:

- Update Client
 - Add the client certificate/keys. Rolling restart all clients so they are aware of the new certificates when asked.
- Update Server
 - Change server configuration to include [`ca-file`](/docs/reference/configuration#ca-file) or [`ca-path`](/docs/reference/configuration#ca-path)
 - Update [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) to `any` or a user 
 defined string representing a tls name to be validated (if validation is required).
 - Rolling restart each of the node.
 - As each node joins the cluster back, the newly established TLS connections will require the clients to present their certificates.

### Upgrade intra node connections to TLS

Rolling upgrades from non-TLS to TLS for intra node connections is supported. The upgrade happens in two rounds:

- Step 1: Enable TLS for heartbeat and fabric across all nodes in a rolling fashion. 
 - Keep non-TLS ports enabled while doing so. 
 - This first step results in a full cluster supporting both, non-TLS as well as TLS. 

{{#info}}
 When upgrading from a version prior to 3.13.0.10 or 3.14.1.9 (which do not support intra node TLS), first upgrade all nodes to 
 version 3.15 and then enable TLS for heartbeat and fabric in a rolling fashion. Nodes running an older version  
 will try connecting to the TLS port first and will fail causing the cluster to not properly form. This will cause warnings to be 
 printed in the log file for each fabric connection attempt. Warning message will look as such:<br>
 `WARNING (tls): (tls_ee.c:277) SSL_accept failed: error:1408F10B:SSL routines:`<br>
 `SSL3_GET_RECORD:wrong version number`<br>
 `WARNING (fabric): (fabric.c:1400) fabric TLS server handshake with 10.10.1.1:50001 failed`
 Versions 3.13.0.10 and 3.14.1.9 do support such upgrade due to the following improvement: <br>
 [AER-5826] - (CLUSTERING) Support rolling upgrades to later versions in which fabric TLS is enabled.
{{/info}}

{{#note}}
 Under normal circumstances, such cluster configured with both non-TLS and TLS fabric/heartbeat will default to TLS. However, downgrade attacks are possible, where an attacker prevents the cluster from establishing TLS connections forcing the cluster to fall back to non-TLS.
{{/note}}

- Step 2: Disable non-TLS for heartbeat and fabric across all nodes in a rolling fashion. 
 - At this point, the cluster only supports TLS connection for its node to node communication. This prevents downgrade attacks.



### Upgrade XDR connections to TLS

For an existing deployment with XDR which will be upgraded to TLS, please follow these steps

- Upgrade Destination Clusters
 - For each one of the destination cluster, Follow the procedure in "Upgrade client to cluster connections to TLS" to enable TLS ports.
- Upgrade Source Cluster
 - Then for each Source Cluster node, upgrade XDR's DC configuration to use the newly configured TLS port.
- Shutdown clear ports (optional)
 - An additional Rolling Restart is required to shutdown destination cluster nodes's clear port (port 3000).


## Examples

### Using cluster-name Standard Authentication

Server Configuration
```
service {
   ...
   cluster-name as-cluster-west
}
 
network {

    tls <cluster-name> {
        cert-file   /home/citrusleaf/x509_certificates/chainless_cluster_chain.pem
        key-file    /home/citrusleaf/x509_certificates/Chainless_cluster/key.pem    
    }

    service {
        tls-port 4333
        tls-name <cluster-name>
        tls-authenticate-client false
        ...
    }
 
    heartbeat {
        ...
        protocol v3
    }
}
```

C Client Benchmark Call

```bash

target/benchmarks -h "192.168.113.203:as-cluster-west:4333" --tlsEnable --tlsCaFile ~/x509_certificates/Platinum/cacert.pem

```

Java Client Benchmark Call

```bash
java -Djavax.net.ssl.trustStore=<TrustStorePath> -Djavax.net.ssl.trustStorePassword=<TrustStorePassword> -jar target/aerospike-benchmarks-*-jar-with-dependencies.jar -h "192.168.113.203:as-cluster-west:4333" -tlsEnable
```

### Using Hostname Match with Common Server Certificate, Mutual Authentication (without name validation), and Client-Certificates Blacklisting

Server Configuration

```
network {


    tls <hostname> {

        ca-file      /home/citrusleaf/x509_certificates/Platinum/cacert.pem  # Root CA's cert
 
        cert-file    /home/citrusleaf/x509_certificates/multi_chain.pem
        key-file     /home/citrusleaf/x509_certificates/MultiServer/key.pem

        cert-blacklist /root/certs_subject/blacklist.txt
    }

    service {
        tls-port 4333
        tls-name <hostname>
        tls-authenticate-client any
        ...
    }

}
```

C Client Benchmark Call

```bash

target/benchmarks -h "192.168.113.203:t3.t-cluster.aerospike.com:4333" --tlsEnable --tlsCaFile ~/x509_certificates/Platinum/cacert.pem --tlsChainFile ~/x509_certificates/client_chain.pem --tlsKeyFile ~/x509_certificates/Client/key.pem

```

Java Client Benchmark Call

```bash

java -Djavax.net.ssl.trustStore=<TrustStorePath> -Djavax.net.ssl.trustStorePassword=<TrustStorePassword> Djavax.net.ssl.keyStore=<KeyStorePath> -Djavax.net.ssl.keyStorePassword=<KeyStorePassword> -jar target/aerospike-benchmarks-*-jar-with-dependencies.jar -h "192.168.113.203:t-cluster.aerospike.com:4333" -tlsEnable

```

### Using Hostname Match with Individual Server Certificate, and Server-Certificates Blacklisting

Server Configuration

```
network {

    tls <hostname> {
 
        cert-file    /home/citrusleaf/x509_certificates/server1_chain.pem  
                     # note - different for each server node.
        key-file     /home/citrusleaf/x509_certificates/Server1/key.pem    
                     # note - different for each server node.
    }

    service {
        tls-port      4333
        tls-name      <hostname>
        tls-authenticate-client false
 
    }
}
```

C Client Benchmark Call

```bash

target/benchmarks -h "192.168.113.203:t3.t-cluster.aerospike.com:4333" --tlsEnable --tlsCaFile ~/x509_certificates/Platinum/cacert.pem --tlsCertBlackList ~/bad_serials.txt

```

Java Client Benchmark Call

```bash

java -Djavax.net.ssl.trustStore=<TrustStorePath> -Djavax.net.ssl.trustStorePassword=<TrustStorePassword> -jar target/aerospike-benchmarks-*-jar-with-dependencies.jar -h "192.168.113.203:t-cluster.aerospike.com:4333" -tlsEnable -tlsRevoke 0xfaa7f3e06e288466c5420e0c7ccd4c45

```

## TLS related configuration parameters

Here is a list of the TLS related configuration parameters for easy reference access:

- To be used in the network's [`tls`](/docs/reference/configuration#tls) sub-stanza:
 - [`cert-file`](/docs/reference/configuration#cert-file)
 - [`key-file`](/docs/reference/configuration#key-file)
 - [`ca-file`](/docs/reference/configuration#ca-file)
 - [`ca-path`](/docs/reference/configuration#ca-path)
 - [`cipher-suite`](/docs/reference/configuration#cipher-suite)
 - [`cert-blacklist`](/docs/reference/configuration#cert-blacklist)
 - [`protocols`](/docs/reference/configuration#protocols)


- To be used in the network's service, fabric, heartbeat sub-stanzas and xdr stanza:
 - [`tls-name`](/docs/reference/configuration#tls-name) 
 - [`tls-port`](/docs/reference/configuration#tls-port) (service, fabric and heartbeat sub-stanzas only)
 - [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) (service sub-stanza only)
 - [`tls-address`](/docs/reference/configuration#tls-address) (service, fabric and hearbeat sub-stanzas only)
 - [`tls-mesh-seed-address-port`](/docs/reference/configuration#tls-mesh-seed-address-port) (heartbeat sub-stanza only)
 - [`tls-node`](/docs/reference/configuration#tls-node) (xdr stanza only)

