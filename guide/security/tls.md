---
title: Transport Layer Security (TLS)
description: TLS feature in Aerospike Enterprise Edition.
assets: /docs/guide/assets
---
Aerospike Enterprise Edition supports Transport Layer Security (TLS) for the following network traffic:

- Client-to-Cluster - Traffic between clients and all Database Nodes (as of version 3.11).
- Cluster-to-Cluster - Traffic between clusters (cross data-center replication through XDR) (as of version 3.11).
- Node-to-Node - Traffic between nodes in a cluster, both fabric and heartbeat (as of version 3.15).

{{#warn}}
This page covers Aerospike Enterprise Edition versions 3.15 and above. Configuration syntax for TLS is different for versions 3.11 to 3.14. Refer to the [TLS Guide pre 3.15](/docs/guide/security/pre3_15/tls_pre3_15.html) and the [Network Configuration/TLS/Pre 3.15](/docs/operations/configure/network/tls/pre3_15) page for details for those specific versions.
{{/warn}}

Aerospike Server's TLS implementation depends on openssl.

- **Clients:** Aerospike Clients's TLS implementation is specific to the language. Refer to [Client Support](/docs/guide/security/tls.html#client-support) for details.

- **Server configuration:** refer to the [Network Configuration/TLS](/docs/operations/configure/network/tls) page for details on configuring TLS.

{{#note}}Description of TLS and certificate signing is out of scope of this documentation.{{/note}}

## TLS Between Clients and Cluster

Aerospike offers 3 modes for TLS Security between clients and a cluster, selected by the [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) configuration parameter in the service sub-stanza:

- Standard Authentication (value `false`)
- Mutual Authentication (value `any`)
- Mutual Authentication with subject name validation (value is user defined, for example `host.domain.com`)

{{#note}}[`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) replaces the 
[`tls-mode`](/docs/reference/configuration#tls-mode) configuration parameter as of version 3.15.{{/note}}

### 1- Standard Authentication

Standard Authentication dictates how the client authenticates each of the node in the Aerospike Cluster via TLS.

#### tls-name and Authentication

The tls-name is used by the client to authenticate each TLS socket connection against a server node, based on the certificate presented by the Aerospike Server node during the initial connection handshake. tls-name for a node is typically the node's hostname. In a typical connection flow:

- The expected tls-name is passed in for the seed node(s).
- For each TLS connection to the seed node, the tls-name match algorithm is executed against the certificate presented by the seed node.
- Once matched, the client fetches from the seed node all peer nodes' connection information and each of their tls-name information, and continues TLS connections to each of the remaining peer nodes.

The tls-name match algorithm is as follows:

- tls-name matches the Subject/CN field.
- tls-name matches the X509v3 Subject Alternative Name.

#### Typical Authentication Setup

Below are some common Standard Authentication setups that a deployment may choose to adapt. Aerospike recommends using the cluster name match setup. 

The common server configuration parameters are the following:

- [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) configuration parameter must be set to `false`.
- [`cert-file`](/docs/reference/configuration#cert-file) must be set. It is the all-in-one .pem file, which includes the server's own certificate. If the certificate is chain-signed, it must also include the chained CA Certificate files in certificate chain order.
- [`key-file`](/docs/reference/configuration#key-file) must be set. It is the .pem file with the private key associated with the certificate.

The client must have the relevant root CA certificate configured.

**A. Hostname Match with Individual Server Certificate**

- Server Certificate
  - Each Aerospike server nodes uses its own cert.
  - The certificate must have the node's system hostname defined in the Subject/CN field. For example:
```
Subject: C=US, ST=California, L=San Jose, O=Acme, Inc., OU=Aerospike Cluster, CN=m1.acme.com
```

- Server Configuration
 - All Aerospike nodes must configure their system's hostname correctly and the name must match what's presented in the in the Subject/CN field.
 - A `tls` sub-stanza for `<hostname>` has to be defined in the aerospike.conf file and the `tls-name` in the service sub-stanza must be specified and have the value `<hostname>` in order for the system's hostname to be read as the tls-name.
- Client Configuration
 - On client applications, for each seed host passed into the cluster_create() call, pass in the same hostname/tls-name as the tls-name.

**B. Hostname Match with Common Server Certificate**

- Server Certificate
 - All Aerospike server nodes use same cert.
 - The certificate must include the x509v3 "Subject Alternate Name" extension, and include the list of all node's hostname in the field. For example:
```
X509v3 extensions:
    X509v3 Subject Alternative Name:
    DNS:m1.acme.com, DNS:m2.acme.com, DNS:m3.acme.com
```

- Server Configuration
 - All Aerospike nodes must configure their system's hostname correctly and the name must match what's presented in the SAN extension.
 - A `tls` sub-stanza for `<hostname>` has to be defined in the aerospike.conf file and the `tls-name` in the service sub-stanza must be specified and have the value `<hostname>` in order for the system's hostname to be read as the tls-name.

- Client Configuration
 - On client applications, for each seed host passed into the cluster_create() call, pass in the same hostname/tls-name as the tlsname.

**C. Wildcard Hostname Match**

- Server Certificate
 - All Aerospike server nodes use same cert.
 - The certificate must have a matcheable wildcard in Subject/CN field. For example:
```
Subject: C=US, ST=California, L=San Jose, O=Acme, Inc., OU=Aerospike Cluster, CN=*.aerospike-cluster.acme.com
```

- Server Configuration
 - All Aerospike nodes must configure their system's hostname correctly to follow the wild card rule.
 - A `tls` sub-stanza for `<hostname>` has to be defined in the aerospike.conf file and the `tls-name` in the service sub-stanza must be specified and have the value `<hostname>` in order for the system's hostname to be read as the tls-name.

- Client Configuration
 - On client applications,  for each seed host passed into the cluster_create() call, pass in the node's hostname as the tlsname.
 - Note - Wildcard hostname match is not available with the Java Client.

**D. Cluster name Match**

- Server Certificate
 - All Aerospike server nodes use same cert.
 - The certificate must have cluster-name defined in the Subject/CN field. For example:
```
Subject: C=US, ST=California, L=San Jose, O=Acme, Inc., OU=Aerospike Cluster, CN=as-cluster-west
```
- Server Configuration
 - All Aerospike server nodes use the same aerospike.conf file, with the same cluster-name specified in the aerospike.conf file.
 - A `tls` sub-stanza for `<cluster-name>` has to be defined in the aerospike.conf file and the `tls-name` in the service sub-stanza must be specified and have the value `<cluster-name>` in order for the node's configured cluster-name to be used as the tls-name.
 - Using cluster-name requires using Aerospike's heartbeat-v3 protocol. See Knowledge Base article.

- Client Configuration
 - On client applications, for each seed host passed into the cluster_create() call, pass in the same cluster-name as the tlsname.

### 2- Mutual Authentication

Mutual authentication requires first deciding on one of the above Standard Authentication setup to allow client authentication of server nodes.

In addition, for the server to authenticate the client, the following is required:

- Client Certificate
 - The certificate must be a valid certificate chain, and must be signed by the same certificate authority as the server.
 - No additional name matching is done.
- Server Configuration
 - [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) configuration parameter must be set to `any` (default).
 - [`ca-file`](/docs/reference/configuration#ca-file)/[`ca-path`](/docs/reference/configuration#ca-path) must be configured to point to the client's signing CA cert.

{{#note}}There can be more than one `tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) directive in order to accept 
multiple subject names. At typical example would be to use different subject names between local clients and XDR clients.
{{/note}}

### 3- Mutual Authentication with subject name validation

This is similar to Mutual Authentication, the only difference being that the name matching would be done.

- Client Certificate
 - The certificate must be a valid certificate chain, and must be signed by the same certificate authority as the server.
 - Name matching is done.
- Server Configuration
 - [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) configuration parameter must be set to the specific name (for example `host.domain.com`). This is the TLS name a cluster node expects clients to present on incoming client connections.
 - [`ca-file`](/docs/reference/configuration#ca-file)/[`ca-path`](/docs/reference/configuration#ca-path) must be configured to point to the client's signing CA cert.


## TLS Between Cluster Nodes

For intra-cluster traffic (both fabric and heartbeat), all cluster nodes share the same TLS name and can thus use the same certificate. Even though intra-cluster TCP connections happen between equal peers, in terms of the TLS handshake between the cluster nodes, the connecting node plays the TLS client role and the accepting node plays the TLS server role. Therefore, the following applies: 

- TLS authentication between cluster nodes is always mutual. The TLS client authenticates the server and vice versa. In contrast to client-facing TLS, this isn't optional or configurable.
- The subject name in a certificate is always validated. A TLS client with a given TLS name expects the accepting TLS server to present a certificate with the same subject name. A TLS server with TLS name X expects the connecting TLS client to present a certificate with subject name X. This means that all cluster nodes need to use the same TLS name.


## TLS Between XDR Replicated Clusters

TLS for XDR traffic is similar to the client-server TLS implementation:

- The source cluster is the "client", thus must have all relevant configurations a TLS client requires, for each DC:
 - [`tls-node`](/docs/reference/configuration#tls-node) specifying the seed-node's ip, tls-name (of remote DC) and port.
 - [`tls-name`](/docs/reference/configuration#tls-name) (of local DC).
- The destination cluster is the "server", thus must have all relevant configurations a TLS server requires (The "service" section must be correctly set up to receive TLS traffic).

## TLS Name Clarification

Each TLS connection has two TLS names, one on each end of the connection. Or, in TLS speak, one for the client (i.e., the end that initiated the TLS connection) and one for the server (i.e., the end that accepted the TLS connection). More precisely, "having a TLS name of X" means "presenting a certificate issued to a subject name of X". If a TLS server has a TLS name of server.aerospike.com, it will present a certificate issued to server.aerospike.com when a TLS client connects to it. Conversely, if a TLS client has a TLS name client.aerospike.com, it will present a certificate issued to client.aerospike.com to a TLS server that it connects to.

As mentioned above, all cluster nodes share the same TLS name for intra-cluster connections (heartbeat and fabric). Therefore, a cluster node will always use the same TLS name (i.e. present the same certificate), regardless of which end of the connection it is on. Only one TLS name needs to be specified for heartbeat and fabric. This is done via the [`tls-name`](/docs/reference/configuration#tls-name) directive.

For the client service and for XDR this is different. Clients are allowed to use a different TLS name than the cluster they connect to. Similarly, an XDR remote DC being replicated to from a local DC is allowed to use a different TLS name than the local DC. This requires the ability to specify two TLS names. In these cases, the [`tls-name`](/docs/reference/configuration#tls-name) directive specifies the local TLS name (the TLS name that a cluster node presents on incoming client connections or on outgoing XDR connections). This covers the first of the two TLS names. The TLS name that a cluster node expects clients to present on incoming client connections is specified via [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client). The TLS name that a cluster node expects the remote DC to present on XDR connections is specified via [`tls-node`](/docs/reference/configuration#tls-node).

In summary:
- Heartbeat and fabric: [`tls-name`](/docs/reference/configuration#tls-name) (for both sides of the TLS connection, client and server)
- Client service: [`tls-name`](/docs/reference/configuration#tls-name) (server) and [`tls-authenticate-client`](/docs/reference/configuration#tls-authenticate-client) (client)
- XDR: [`tls-name`](/docs/reference/configuration#tls-name) (local DC) and [`tls-node`](/docs/reference/configuration#tls-node) (remote DC)


## Client Support

- C-Client - uses OpenSSL.
 - Note - Asynchronous libuv API does not support TLS.
- Java Client - uses Java's built-in security library, javax.net.ssl.SSLSocket, and java.security.cert.X509Certificate.
 - Note - Asynchronous API supports TLS only when using the netty async framework (as of version [4.1.10](https://www.aerospike.com/download/client/java/notes.html#4.1.10))
 - Note - Wildcard match not supported.
 - For running with TLS, a java application must have the appropriate certificate added to the local truststore (for standard authentication) and local keystore (for mutual authentication). When required, the relevant script should be modified to supply the location of the truststore and keystore.
- C# Client - uses C#'s built-in security library, System.Security.Authentication.
 - Note - Asynchronous API does not support TLS.
 - Note - Wildcard match not supported.
- Ruby Client - uses the [openssl](https://rubygems.org/gems/openssl) gem, which wraps the native OpenSSL library.

## Revoking Rogue Certificates

Aerospike provides an easy way to filter out rogue certificates through a black-list mechanism.
If a certificate should no longer be legitimate, it should be added to the appropriate blacklist.

Refer to the [`cert-blacklist`](/docs/reference/configuration#cert-blacklist) configuration parameter for further details.

## TLS version and Cipher Choice

Aerospike server defaults to TLS1.2. This can be overridden through configurations both on client and server side.

Aerospike relies on the TLS handshake to determine optimal cipher suite to pick, with server as the choice preference. It is also possible to explicitly set the cipher choice through configurations both on client and server side.

Refer to the [`cipher-suite`](/docs/reference/configuration#cipher-suite) configuration parameter for further details.

