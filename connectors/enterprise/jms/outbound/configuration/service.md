---
title: Aerospike Connect for JMS - Outbound Service Configuration
description: Configuring Aerospike Connect for JMS Outbound Connector service section
---

## Service

The service section configures the connector's listening ports, TLS and
network interface.

The following options are available:

Option | Required | Default | Description
---| --- | --- | ---
port | no	|8080	| The port the server listens to. |
address |	no	| 0.0.0.0	| The interface ip address the server binds to. Use 0.0.0.0 for all interfaces. |
tls | Required if port not specified. | | Tls specific server configuration described [below](#tls-configuration). |
http2-max-concurrent-streams |	no	| No limit. | Maximum concurrent streams for http2 connections. No limit if this value is not set or <= 0.|

### TLS configuration

The service -> tls section configures server side TLS for the JMS outbound server.

The configuration options are:

Option | Required | Default | Description
---| --- | --- | ---
port | no	|	| The HTTPS/TLS port the server  listens to.|
address |	no	| 0.0.0.0	| The interface ip address the TLS server binds to. Use 0.0.0.0 for all interfaces. |
key-store | no | | The [keystore](#tls-store) configuration containing server side certificate and key.|
trust-store | no | Default java trust store. | The [keystore](#tls-store) configuration containing trusted CA certificates.|
protocols | no | TLSv1.2 | List of allowed TLS protocols.  |
ciphers | no | Default java ciphers  | List of allowed ciphers. |
allowed-peer-names | no | | List of client (aerospike server nodes) peer names for mutual authentication. If set, only those clients (aerospike server nodes) that present certificates matching the peer names will be allowed to connect. When using HTTPv2, mutual authentication requires, server to run with Java 11+ to use TLSv1.3.|

#### TLS Store
A tls key/trust store configuration. All relative file paths are considered relative to the configuration file directory.
Refer to [creating tls keystores](/docs/connectors/create-tls-keystores.html) page for creating key and trust stores.

Option | Required | Default | Description
---| --- | --- | ---
store-file | yes | | The store file. |
store-password-file | yes | | Read store password from this file.|
key-password-file | no | | Read key password from this file. |
store-type | no | JKS | The keystore type. Valid values are JKS, JCEKS, PKCS12, PKCS11, DKS, Windows_MY, BKS. |

### Examples

#### Clear text only

```
service:
  port: 8080
  address: 192.168.5.154
```

#### TLS only
```
service:
  tls:
    port: 8443
    allowed-peer-names:
      - asd.aerospike.com
    protocols:
      - tlsv1.3
    trust-store:
      store-file: tls/ca.aerospike.com.truststore.jks
      store-password-file: tls/storepass
    key-store:
      store-file: tls/connector.aerospike.com.keystore.jks
      store-password-file: tls/storepass
      key-password-file: tls/keypass
```

#### Clear test and TLS
```
service:
  port: 8080
  address: 192.168.5.154
  tls:
    port: 8443
    allowed-peer-names:
      - asd.aerospike.com
    protocols:
      - tlsv1.3
    trust-store:
      store-file: tls/ca.aerospike.com.truststore.jks
      store-password-file: tls/storepass
    key-store:
      store-file: tls/connector.aerospike.com.keystore.jks
      store-password-file: tls/storepass
      key-password-file: tls/keypass
```
