---
title: TLS for versions prior to 3.15
description: TLS feature in Aerospike Enterprise Edition for versions prior to 3.15.
assets: /docs/guide/assets
---
Aerospike Enterprise Server Version 3.11 introduced Transport Layer Security for the following network traffic:

- Client-to-Server - Traffic between clients and all Database Nodes.
- Cluster-to-Cluster - Traffic between data-center replication (XDR).

{{#warn}}
This page covers Aerospike Enterprise Edition versions 3.11 to 3.14. Configuration syntax for TLS is different for versions 3.15 and above. Refer to the [TLS Guide](/docs/guide/security/tls.html) and the [Network Configuration/TLS](/docs/operations/configure/network/tls) page for details for versions 3.15 and above.
Also, in version 3.15, Aerospike Enterprise Edition introduced Node-to-Node (Fabric and Heartbeat) network traffic Transport Layer Security. 
{{/warn}}


Aerospike Server's TLS implementation depends on openssl.

Aerospike Clients's TLS implementation is specific to the language. Please see [Client Support](/docs/guide/security/tls.html#client-support).

For details on configuring TLS in Aerospike, refer to [Network Configuration/TLS](/docs/operations/configure/network/tls) (versions 3.15 and above) or [Network Configuration/TLS/Pre 3.15](/docs/operations/configure/network/tls/pre3_15) (versions 3.11 to 3.14).

Note - Description of TLS and certificate signing is out of scope of this documentation.

## TLS Security Mode

Aerospike Offers 3 optional modes for TLS Security. Each will be described in detail.

- Standard Authentication
- Mutual Authentication
- Encryption Only

This choice is dictated by the new server configuration parameter tls-mode.

### Standard Authentication

Standard Authentication dictates how the client authenticates each of the node in the Aerospike Cluster via TLS.

#### tls-name and Authentication

The tls-name is used by the client to authenticate each TLS socket connection against a server node, based on the certificate presented by the Aerospike Server node during the initial connection handshake. tls-name for a node is typically the node's hostname. In a typical connection flow,

- The expected tls-name is passed in for the seed node(s).
- For each TLS connection to the seed node, the tls-name match algorithm is executed against the certificate presented by the seed node.
- Once matched, the client fetches from the seed node all peer nodes' connection information and each of their tls-name information, and continues TLS connections to each of the remaining peer nodes.

The tls-name match algorithm is as follows -

- tls-name matches the Subject/CN field.
- tls-name matches the X509v3 Subject Alternative Name.

#### Typical Authentication Setup
Next, we specify some common Standard Authentication setups that a deployment may choose to adapt. Aerospike recommends using the Cluster-Name Match setup. 

The common server configuration parameters are the following -

- tls-mode must be set to authenticate-server. (default).
- tls-certfile must be set. It is the all-in-one .pem file, which includes the server's own certificate. If the certificate is chain-signed, it must also include the chained CA Certificate files in certificate chain order.
- tls-keyfile must be set. It is the .pem file with the private key associated with the certificate.

The client must have the relevant root CA certificate configured.

#### Hostname Match with Individual Server Certificate

- Server Certificate
  - Each Aerospike server nodes uses its own cert.
  - The certificate must have the node's system hostname defined in the Subject/CN field. For example,

```
Subject: C=US, ST=California, L=San Jose, O=Acme, Inc., OU=Aerospike Cluster, CN=m1.acme.com
```

- Server Configuration
 - All Aerospike nodes must configure their system's hostname correctly and the name must match what's presented in the in the Subject/CN field.
 - tls-name must be specified in the aerospike.conf file, and must have the value `<hostname>` so the system's hostname will be read as the tls-name.
- Client Configuration
 - On client applications, for each seed host passed into the cluster_create() call, pass in the same hostname/tls-name as the tls-name.

#### Hostname Match with Common Server Certificate

- Server Certificate
 - All Aerospike server nodes use same cert.
 - The certificate must include the x509v3 "Subject Alternate Name" extension, and include the list of all node's hostname in the field. For example -

```
X509v3 extensions:
    X509v3 Subject Alternative Name:
    DNS:m1.acme.com, DNS:m2.acme.com, DNS:m3.acme.com
```

- Server Configuration
 - All Aerospike nodes must configure their system's hostname correctly and the name must match what's presented in the SAN extension.
 - tls-name MUST be specified in the aerospike.conf file, and must have the value '<hostname>' so the system's hostname will be read as the tls-name.

- Client Configuration
 - On client applications, for each seed host passed into the cluster_create() call, pass in the same hostname/tls-name as the tlsname.

#### Wildcard Hostname Match

- Server Certificate
 - All Aerospike server nodes use same cert.
 - The certificate must have a matcheable wildcard in Subject/CN field. For example -

```
Subject: C=US, ST=California, L=San Jose, O=Acme, Inc., OU=Aerospike Cluster, CN=*.aerospike-cluster.acme.com
```

- Server Configuration
 - All Aerospike nodes must configure their system's hostname correctly to follow the wild card rule.
 - tls-name MUST be specified in the aerospike.conf file, and MUST be <hostname> so the system's hostname will be read as the tls-name.

- Client Configuration
 - On client applications,  for each seed host passed into the cluster_create() call, pass in the node's hostname as the tlsname.
 - Note - Wildcard hostname match is not available with the Java Client.

#### Cluster-name Match

- Server Certificate
 - All Aerospike server nodes use same cert.
 - The certificate must have cluster-name defined in the Subject/CN field. For example,
```
        Subject: C=US, ST=California, L=San Jose, O=Acme, Inc., OU=Aerospike Cluster, CN=as-cluster-west
```
- Server Configuration
 - All Aerospike server nodes use the same aerospike.conf file, with the same cluster-name specified in the aerospike.conf file.
 - tls-name must be specified in the aerospike.conf file, and must have the value `<cluster-name>` so the node's configured cluster-name will be used as the tls-name.
 - Using cluster-name requires using Aerospike's heartbeat-v3 protocol. See Knowledge Base article.

- Client Configuration
 - On client applications, for each seed host passed into the cluster_create() call, pass in the same cluster-name as the tlsname.

### Mutual Authentication

Mutual authentication requires first deciding on one of the above Standard Authentication setup to allow client authentication of server nodes.

In addition, for the server to authenticate the client -

- Client Certificate
 - All client nodes use same certificate.
 - The certificate must be a valid certificate chain, and must be signed by the same certificate authority as the server.
 - No additional name matching is done.
- Server Configuration
 - tls-mode must be set to authenticate-both.
 - tls-cafile/tls-capath must be configured to point to the client's signing CA cert.

### Encryption-Only
In some deployments, only transport-level encryption is required. 

- Certificates
 - No certificates are required on either server or client side.
- Server Configuration
 - tls-mode must be set to encrypt-only.
- Client Configuration
 - Must have configuration encrypt-only.
 - Note - this is not supported in C#, as C# does not support anonymous ciphers.

## Revoke Rogue Certificates

Aerospike provides an easy way to filter out rogue certificates through a black-list mechanism.
If a certificate should no longer be legitimate, it should be added to the appropriate blacklist.

## TLS version and Cipher Choice

Aerospike server defaults to TLS1.2 and above to ensure the best protocol is used. This can be overridden through configurations both on client and server side.

Aerospike relies on the TLS handshake to determine optimal cipher suite to pick, with server as the choice preference. It is also possible to explicitly set the cipher choice through configurations both on client and server side.

## XDR

TLS for XDR traffic is very similar to client-server TLS implementation -

- The source cluster is the "client", thus must have all relevant configurations a TLS client requires, for each DC -
 - seed-node's ip, tls-name, and port
 - tls-cafile
- The destination cluster is the "server", thus must have all relevant configurations a TLS server requires.
 - The "service" section correctly set up to receive TLS traffic.
 - Note - For XDR traffic, "encrypt-only" is not supported.

## Client Support

- C-Client - uses OpenSSL.
 - Note - Asynchronous libuv API does not support TLS.
- Java Client - uses Java's built-in security library, javax.net.ssl.SSLSocket, and java.security.cert.X509Certificate.
 - Note - Asynchronous API does not support TLS
 - Note - Wildcard match not supported.
 - For running with TLS, a java application must have the appropriate certificate added to the local truststore (for standard authentication) and local keystore (for mutual authentication). When required, the relevant script should be modified to supply the location of the truststore and keystore.
- C# Client - uses C#'s built-in security library, System.Security.Authentication.
 - Note - Asynchronous API does not support TLS.
 - Note - Wildcard match not supported.
 - Note - Does not support encryption-only since no anonymous ciphers support.
