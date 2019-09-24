---
title: Connecting
description: Use the Aerospike Ruby client object to connect and periodically ping nodes for cluster status. 
---

Use the Aerospike Ruby `Client` object to connect to a cluster of Aerospike
server nodes. First, instantiate a new `Client` object to specify the IP address
and port of one or more cluster seed nodes. The client will discover the
remaining cluster nodes automatically. It will periodically ping all nodes for
cluster status in a background thread it creates.

The `Client` instance is thread-safe and can be used concurrently. Each `get`
or `set` call is a non-blocking, asynchronous network call to the Aerospike
database cluster. Connections are cached in a connection pool for each server
node.

### Setting Seed Nodes via Hosts String

Create a new `Client` object to specify the server connection using the IP
address and port. The client makes initial contact to the specified server
node, and then discovers all other cluster nodes.

The hosts string can contain a single address or a comma-separated list of node
addresses. The port number is optional and defaults to 3000 if not specified.

```ruby
client = Aerospike::Client.new("10.0.0.10:3000,10.0.0.11:3000")
```

If no hosts string is passed in the Client initializer, the client will attempt
to read the hosts from the `AEROSPIKE_HOSTS` environment variable.

### Setting Multiple Seed Nodes via Host Array

Alternatively, instead of passing the list of seed nodes as a hosts string, the
addresses of the seed nodes can be passed as an array of `Host` instances.
To connect to any known cluster node, specify each node in the cluster when
creating the `Client` object. The client iterates through the node array until
successfully connecting to a node. It then discovers all other cluster nodes.

```ruby
hosts = [
  Host.new("a.host", 3000),
  Host.new("another.host", 3000),
  Host.new("yet.another.host", 3000),
]

client = Client.new(hosts)
```

### IPv6

Aerospike Enterprise Edition version 3.10 and later support IPv6. Please refer
to the [IPv6 Configuration](/docs/operations/configure/network/ipv6/index.html)
section in the operations manual for how to configure your cluster to use IPv6.
To connect to a cluster with the Ruby client using IPv6, specify the IPv6
address of one of more cluster nodes as the host seed address(es). Note that on
the client side, when using IPv6 addresses in a hosts string, the IPv6
addresses must be enclosed in square brackets, e.g.
`[fde4:8dba:82e1::c4]:3000`.

```ruby
client = Aerospike::Client.new("[fde4:8dba:82e1::c4]:3000")
```

### Setting the Client Policy

The client policy can be set in the client initializer. The policy controls
parameters of the client's overall behavior like e.g. connectiton timeout
limits. The client policy can also set default values for the client's
read/write/query/scan policies. These policy values affect specific client
commands. The defautl policy values can be overridden when a command is issued
to the client.

```ruby
policy = Aerospike::ClientPolicy.new(timeout: 1.0)
client = Aerospike::Client.new("127.0.0.1", policy: policy)
client.connect()

result = client.get(Aerospike::Key.new("test", "test", "demo"), nil, {timeout: 0.5})
```

### TLS Encrypted Connections

Starting with Aerospike Enterprise Edition version 3.11, the server supports
Transport Layer Security (TLS) encryption for secure connections between the
clients and the cluster nodes. Please refer to the [TLS
Guide](/docs/guide/security/tls.html) for more information on this feature.

To connect to Aerospike cluster using TLS, the client policy needs to be
configured correctly by passing an `ssl_options` hash with one or more of the
following properties:

- `ca_file` - The path to a file containing a PEM-format CA certificate.
- `ca_path` - The path to a directory containing CA certificates in PEM format. Files are looked up by subject's X509 name's hash value.
- `cert_file` - The path to a file containing a PEM-format client certificate.
- `pkey_file` - The path to a file containing a PEM-format private key for the client certificate.
- `pkey_pass` - Optional password in case `pkey_file` is an encrypted PEM resource.

These parameters are used to setup an
[`SSLContext`](https://ruby.github.io/openssl/OpenSSL/SSL/SSLContext.html).
Alternatively, a pre-configured `SSLContext` can be passed using the `context`
property of `ssl_options`. In this case, all other properties in `ssl_options`
are ignored.

The Aerospike server typically listens on two separate ports for unencrypted
and TLS-encrypted connections. When using TLS, ensure that the client's seed
nodes are configured with the correct port number.

To facilitate verification of the server certificate, the client also needs to
know the server's tls-name. The expected tls-name needs to be set in the seed
node configuration. When using a hosts string (either passed via the Client
constructor or the `AEROSPIKE_HOSTS` env variable) each seed node address needs
to be specified as

    <hostname or IP> [ ":" <tls-name> ] [ ":" <port> ]

If not specified, the tls-name defaults to the cluster name, if configured, or
else the node's hostname.

#### Examples

Enabling TLS encryption by specifying the cert/private key file:

```ruby
require 'aerospike'
include Aerospike

tls_options = {
  ca_file = './tls/ca.cert.pem',
  cert_file = './tls/client.cert.pem',
  pkey_file = './tls/client.key.pem'
}
policy = ClientPolicy.new(tls: tls_options)
seed = '10.0.0.10:mydb:3000'

client = Client.new(seed, policy: policy)
```

Enabling TLS encryption by passing a pre-configured `SSLContext`:

```ruby
require 'aerospike'
require 'openssl'

include Aerospike

context = OpenSSL::SSL::SSLContext.new
cert = OpenSSL::X509::Certificate.new(File.read('./tls/client.cert.pem'))
pkey = OpenSSL::PKey.read(File.read('./tls/client.key.pem'))
context.add_certificate(cert, pkey)
context.ciphers = [ ... ] # Customize the context

policy = ClientPolicy.new(tls: { context: context })
seed = '10.0.0.10:mydb:3000'

client = Client.new(seed, policy: policy)
```

### Cleaning Up

When all transactions complete and the application is ready for a clean shutdown, call `#close` to free resources held by the `Client` object: 

```ruby
client.close
```

{{#note}}
The `Client` object cannot be used after `#close` is called.
{{/note}}
