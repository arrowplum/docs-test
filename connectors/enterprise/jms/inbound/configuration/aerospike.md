---
title: Aerospike Configuration
description: Properties specific to Aerospike
---

The Aerospike section configures the connection properties to the Aerospike
cluster.

| Option       | Required | Description                                                                                             |
| ------------ | -------- | ------------------------------------------------------------------------------------------------------- |
| seeds        | yes      | The list of Aerospike seed nodes to connect. See <a href="#seeds-config"> seeds</a>.                    |
| credentials  | no       | The credentials to connect to the Aerospike server. See <a href="#credentials-config"> credentials</a>. |
| services     | no       | The service configuration. See <a href="#services-config"> services</a>.                                |
| cluster-name | no       | The name of the servers in the Aerospike cluster to connect to.                                         |
| performance  | no       | The performance tuning parameters. See <a href="#performance-config"> performance</a>.                  |
| rack-id      | no       | The rack where the connector instance resides.                                                          |
| tls          | no       | The tls config. See <a href="#tls-config"> tls</a>.                                                     |

## Seeds Config

A map of Aerospike seed to its configuration.

| Option   | Required | Default | Description                           |
| -------- | -------- | ------- | ------------------------------------- |
| port     | no       | 3000    | The Aerospike server port.            |
| tls-name | no       |         | The tls name of the Aerospike server. |

## Credentials Config

The credentials to connect to the Aerospike server.

| Option        | Required | Default  | Description                                                                                    |
| ------------- | -------- | -------- | ---------------------------------------------------------------------------------------------- |
| username      | yes      |          | The username.                                                                                  |
| password-file | yes      |          | The file containing the password. See <a href="#password-file"> password file</a> for details. |
| auth-mode     | no       | internal | The authentication mode. Valid values are internal, external, external-insecure.               |

## Services Config

The service configuration.

| Option                 | Required | Default        | Description                                                                                   |
| ---------------------- | -------- | -------------- | --------------------------------------------------------------------------------------------- |
| ip-map                 | no       | no translation | The IP translation table. See <a href="#ip-map-config"> ip map</a>.                           |
| use-services-alternate | no       | false          | Should use "services-alternate" instead of "services" in info request during cluster tending. |

## IP Map Config

A IP translation table is a map of ip address to ip address, used in cases where
different clients use different server IP addresses. The key is the IP address
returned from friend info requests to other servers. The value is the real IP
address used to connect to the server.

## Performance Config

The performance tuning parameters.

| Option                    | Required | Default         | Description                                                         |
| ------------------------- | -------- | --------------- | ------------------------------------------------------------------- |
| max-connections-per-node  | no       | 300             | The maximum number of connections allowed per Aerospike server node |
| connection-pools-per-node | no       | 1               | The number of synchronous connection pools used for each node.      |
| event-loop-size           | no       | # of processors | The event loop size.                                                |

## TLS Config

The tls config of the client.

| Option              | Required | Default                            | Description                                                                                                                          |
| ------------------- | -------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| key-store           | yes      |                                    | The key store containing the Aerospike client certificate for mutual authentication. See <a href="#tls-store-config"> tls store</a>. |
| trust-store         | no       | Default java trust store.          | The trust store containing trusted CA certificate for Aerospike server certificate. See <a href="#tls-store-config"> tls store</a>.  |
| ciphers             | no       | default ciphers allowed by the JVM | Allowed list of TLS ciphers that clients can use for secure connections.                                                             |
| revoke-certificates | no       |                                    | List of certificate serial numbers to reject.                                                                                        |

### Sample TLS section with default trust store

```
  tls:
    trust-store: default
```

## TLS Store Config

A tls key/trust store.

| Option              | Required | Default | Description                                                                          |
| ------------------- | -------- | ------- | ------------------------------------------------------------------------------------ |
| store-file          | yes      |         | The store file                                                                       |
| store-password-file | yes      |         | Read store password from this file.                                                  |
| key-password-file   | no       |         | Read key password from this file.                                                    |
| store-type          | no       | JKS     | The keystore type. Valid values are JKS, JCEKS, PKCS12, PKCS11, DKS, Windows_MY, BKS |

## Example

```
aerospike:
  seeds:
    - 192.168.50.1:
        port: 3000
        tls-name: red
    - 192.168.50.2
  credentials:
    username: admin
    password-file: /path/to/password/file.txt
    auth-mode: internal
  services:
    ip-map:
      192.168.50.1: 192.168.60.1
      192.168.50.2: 192.168.60.2
    use-services-alternate: false
  cluster-name: east
  performance:
    max-connections-per-node: 300
    connection-pools-per-node: 1
    event-loop-size: 4
  rack-id: 1
  tls:
    key-store:
      store-file: /path/to/store/file
      store-password-file: /path/to/store/password/file
      key-password-file: /path/to/key/password/file
      store-type: JKS
    trust-store:
      store-file: /path/to/store/file
      store-password-file: /path/to/store/password/file
      key-password-file: /path/to/key/password/file
      store-type: JKS
    ciphers:
      - TLS_RSA_WITH_3DES_EDE_CBC_SHA
    revoke-certificates:
      - 12345678
```
