---
title: TLS Configuration for versions prior to 3.15
description: Configure cluster to use TLS for versions prior to 3.15. 
---

Since 3.11 Enterprise Release, Aerospike clusters can be configured to use TLS for client-to-cluster connections and Datacenter-to-Datacenter connections.

{{#warn}}
This page covers options and relevant configurations for Aerospike Enterprise Edition versions 3.11 to 3.14. Configuration syntax for TLS is different for versions 3.15 and above. Refer to the [TLS Guide](/docs/guide/security/tls.html) and the [Network Configuration/TLS](/docs/operations/configure/network/tls) page for details for versions 3.15 and above.
Also, in version 3.15, Aerospike Enterprise Edition introduced Node-to-Node (Fabric and Heartbeat) network traffic Transport Layer Security. 
{{/warn}}

If you are planning to migrate your network to a TLS environment, please contact Aerospike Support for a smooth migration.

Please first review the [TLS Guide pre 3.15](/docs/guide/security/pre3_15/tls_pre3_15.html) for an overview of the functionality. 

Then decide:
- If TLS should be run only between your client-server network only, or if you have XDR and DC-to-DC traffic should also be TLS.
- If you require Standard Authentication or Mutual Authentication in each network.
- What type certificate setup you choose - Cluster-name Match, Wildcard Hostname Match, Match with Common Server Certificate, or Hostname Match with Individual Server Certificate.

Once the TLS deployment artichecture is determined, then generate the appropriate certificates, and follow the upgrade scenario below.

## Configuration Parameters Reference

Below lists the relevant configuration parameters for TLS. Please also see [Configuration Reference](/docs/reference/configuration) for additional details on each parameter.

```bash
# (*)  required parameters for TLS
# (**) required parameters when tls-mode is authenticate-server or authenticate-both
# (+)  optional parameters when tls-mode is authenticate-both
 
network {
  service {
    port 3000
 
    tls-port 4333                   # (*) Enables tls
    tls-address 192.168.90.150      # Bind address for TLS
    tls-mode authenticate-server|authenticate-both|encrypt-only 
                                    # If not specified, default is authenticate-server, encrypt-only not supported for XDR.
    tls-name <cluster-name>|<hostname>|user-specific  
                                    # (*) What is being used for authentication. (populated in the tls-name wire field)
    tls-certfile path-to-file       # (**) Conditionally required if authenticate-server or authenticate-both
    tls-keyfile path-to-file        # (**) Conditionally required if authenticate-server or authenticate-both
    tls-cafile path-to-file         # (+) Relevant if authenticate-both. Only one of tls-cafile or tls-capath is required.
    tls-capath directory-path       # (+) Relevant if authenticate-both. Only one of tls-cafile or tls-capath is required.
    tls-protocols  -all,+TLSv1.2    # Protocol version to include
    tls-cipher-suite ALL:!COMPLEMENTOFDEFAULT:!eNULL 
                                    # Ciphers to includes. If encrypt-only mode, must be set to aNULL
    tls-cert-blacklist path-to-file # Blacklist including serial number of certs to revoke
  }
}
 
xdr {
  enable-xdr      true
 
  datacenter dc1 {
     tls-node 10.10.121.100 tls-node1 4000  # (*) Remote seed node`s ip address, tls-name, port-number
     tls-cafile path-to-file           # (+) Only one of tls-cafile or tls-capath is required.
                                       # Generally, this should be set to the system`s default (/etc/ssl/certs/cacert.pem on Ubuntu)
     tls-capath directory-path         # (+) Only one of tls-cafile or tls-capath is required.
                                       # Generally, this should be set to the system`s default (/etc/ssl/certs on Ubuntu)
     tls-certfile <path-to-file>       # (**) Conditionally required if authenticate-both
     tls-keyfile <path-to-file>        # (**) Conditionally required if authenticate-both
     tls-cert-blacklist <path-to-file> # Only loaded on daemon start-up. Can be dynamically refreshed using the tls-cert-blacklist command above.
  }
 
  datacenter dc2 {
    # can have its own set of tls configs
  }
}
```

- tls-name has the ability to be substituted. For example
 - tls-name `<cluster-name>` means the server will use what's defined in cluster-name as the tls-name attribute.
 - tls-name `<hosname>` means the server will use what's returned by the underlying OS's "hostname" system call as the tls-name attribute.
- tls-capath it may be required to run a "c_rehash" to generate symlinks based on hash values.

## Configuring Blacklist

The serial number assumes hex representation, with no non-hex characters such as 'x' or ":" etc.

To add a certificate's information to the blacklist, you must have the certificate available to extract the properly formatted serial number.

```
cat chain.pem | openssl x509 -noout -serial -issuer | \
awk 'match($0,/[^=]*=(.*)/, a) { printf "%s", a[1] } END {printf "\n"}'
```

### Refreshing tls-cert-blacklist

In addition, the xdr blacklist can be refreshed through a new command -

```
asinfo -v 'set-config:context=xdr;dc=REMOTE;tls-cert-blacklist=.../blah/blacklist.txt'
```

## Upgrading to TLS
Below we list some of the common upgrade scenario. 

### Upgrade from Clear network to TLS network

For an existing deployment which will be upgraded to TLS, please follow these steps

- Upgrade Server
 - Rolling Upgrade each node in the cluster to a software version that is TLS aware (3.11 and above), while adding in new TLS enabled configurations. Do not shutdown clear ports.
 - Repeat for all nodes in the cluster.
 - During this time, client will continue to communicate with cluster via clear transport.
- Upgrade Client
 - Once every node is TLS enabled, start upgrading client to a software version that is TLS aware, while adding in new TLS enabled client configurations.
 - During this time, old clients will continue to communicate with cluster via clear transport, while new clients will communicate with cluster via TLS.
 - Repeat for all application nodes.
- (optional) Shut down Clear Ports
 - If no clients should be allowed to reach the server, an additional Rolling Restart is required to remove and shutdown the clear port.

### Upgrade from Standard-Authentication to Mutual-Authentication

For an existing deployment already running standard-authentication and will be upgraded to mutual-authentication, please follow these steps

- Update Client
 - Add the client certificate/keys. Rolling restart all clients so they are aware of the new certificates when asked.
- Update Server
 - Change server configuration to include tls-cafile, and change tls-mode=authenticate-both.
 - Rolling Restart each of the node.
 - As each node starts backup, the newly established TLS connections will require the clients to present their certificates.

### Upgrade XDR

For an existing deployment with XDR which will be upgraded to TLS, please follow these steps

- Upgrade Destination Clusters
 - For each one of the destination cluster, Follow the procedure in "Upgrade from Clear network to TLS network" to enable TLS ports
- Upgrade Source Cluster
 - Then for each Source Cluster node, upgrade XDR's DC configuration to use the newly configured TLS port.
- (optional) Shutdown Clear Ports
 - An additional Rolling Restart is required to shutdown destination cluster nodes's clear port (port 3000).

{{#note}}
Note: 'encrypt-only' is not supported for XDR.
{{/note}}

## Examples

### Using cluster-name Standard Authentication

Server Configuration
```
service {
   ...
   cluster-name as-cluster-west
}
 
network {
    service {
        tls-port 4333
        tls-name <cluster-name>
 
        tls-certfile   /home/citrusleaf/x509_certificates/chainless_cluster_chain.pem
        tls-keyfile    /home/citrusleaf/x509_certificates/Chainless_cluster/key.pem
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

### Using Hostname Match with Common Server Certificate, Mutual Authentication, and Client-Certificates Blacklisting

Server Configuration

```
network {
    service {
        tls-port        4333
        tls-mode        authenticate-both
        tls-name        <hostname>
 
        tls-cafile      /home/citrusleaf/x509_certificates/Platinum/cacert.pem  # Root CA's cert
 
        tls-certfile    /home/citrusleaf/x509_certificates/multi_chain.pem
        tls-keyfile     /home/citrusleaf/x509_certificates/MultiServer/key.pem
 
        tls-cert-blacklist /root/certs_subject/blacklist.txt
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

### Using Hostname Match with Individual Server Certificate,  and Server-Certificates Blacklisting

Server Configuration

```
network {
    service {
        tls-port      4333
 
        tls-name      <hostname>
 
        tls-certfile  /home/citrusleaf/x509_certificates/server1_chain.pem  # note - different for each server node.
        tls-keyfile   /home/citrusleaf/x509_certificates/Server1/key.pem    # note - different for each server node.
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

### Using Encryption-Only


Server Configuration

```
network {
    service {
        tls-port          4333
        tls-mode          encrypt-only
        tls-cipher-suite  aNULL
    }
}
```

C Client Benchmark Call

```bash

target/benchmarks  --tlsEnable --tlsEncryptOnly --tlsCipherSuite aNULL -h "192.168.113.203" -p 4333

```

Java Client Benchmark Call

```bash

java -Djavax.net.ssl.trustStore=<TrustStorePath> -Djavax.net.ssl.trustStorePassword=<TrustStorePassword> -jar target/aerospike-benchmarks-*-jar-with-dependencies.jar -h 192.168.113.203 -p 4333 -tlsEnable -tlsEncryptOnly -tlsCipherSuite TLS_DH_anon_WITH_AES_128_CBC_SHA256

```
