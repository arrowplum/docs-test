---
title: Aerospike Information Tool (asinfo)
description: Learn the details of asinfo, an extensive command-line utility that provides an interface to Aerospike's cluster command and control functions.
---

**asinfo** is a command-line utility that provides an interface to Aerospike's
cluster command and control functions. This includes the ability to change server configuration parameters while the Aerospike is running.
{{#info}}Configuration changes made by asinfo are not persisted to the Aerospike configuration file. If a change requires persistence it needs to be added to the configuration file.{{/info}}

### Usage
This utility is packaged in the Aerospike Tools package and by default installs to `/usr/bin/asinfo`. To execute **asinfo** use the following format. Please remember to put the VALUE in quote ("").
```
asinfo [-V] [-E] [-v VALUE] [-l] [-h HOST] [-p PORT] [-U USER]
       [-P [PASSWORD]] [-t TLS_NAME] [--tls-enable]
       [--tls-cafile TLS-CAFILE] [--tls-capath TLS-CAPATH]
       [--tls-protocols TLS-PROTOCOLS]
       [--tls-cipher-suite TLS-CIPHER-SUITE]
       [--tls-keyfile TLS-KEYFILE] [--tls-certfile TLS-CERTFILE]
       [--tls-cert-blacklist TLS-CERT-BLACKLIST] [--tls-crl-check]
       [--tls-crl-check-all] [--config-file CONFIG-FILE]
       [--instance INSTANCE] [--no-config-file]
       [--only-config-file ONLY-CONFIG-FILE]
```

asinfo can be configured by using tools configuration files. Following summary explains all configuration options. 
Please see [Aerospike Tools Configuration](/docs/tools/conffile) for more details.

| Option | Default | Description |
|--------|---------|-------------|
| -h     | localhost | IP Address or FQDN of the target Aerospike server. |
| -p     | 3000 | Service port of the target Aerospike server. |
| -t     | | TLS name of host to verify for TLS connection. |
| -U     | | Aerospike user name. |
| -P     | | Aerospike user password. |
| -v     | | Command to send to the target server. If not provided returns a default set of results. See [Commands section](#commands) below. |
| -l     | disabled | Replaced semicolons ';' in with line breaks in the response. |
| -V     | disabled | Show the version of asinfo and exit. |
| -E     | disabled | show program usage. |
| --tls-enable     | disabled | Enable TLS on connections. |
| --tls-cafile     | | Path to a trusted CA certificate file. |
| --tls-capath     | | Path to a directory of trusted CA certificates. |
| --tls-protocols     | '-all +TLSv1.2' | Set the TLS protocol selection criteria. |
| --tls-cipher-suite     | | Set the TLS cipher selection criteria. |
| --tls-keyfile     | | Path to the key for mutual authentication. |
| --tls-certfile     | | Path to the chain file for mutual authentication. |
| --tls-cert-blacklist     | | Path to a certificate blacklist file. |
| --tls-crl-check     | | Enable CRL checking for leaf certificate. |
| --tls-crl-check-all     | | Enable CRL checking for entire certificate chain. |
| --timeout     | 5 seconds | Set timeout value in seconds. TLS connection does not support timeout. |
| --config-file     | | Read this file after default configuration file. |
| --instance     | | Section with these instance is read. e.g in case instance `a` is specified section cluster_a is read. |
| --no-config-file     | disabled | Do not read any config file. |
| --only-config-file     | | Read only this configuration file. |

**Example:**

```
$ asinfo -v "namespaces"
requested value  namespaces
value is user_profile;test;bar
```

### Aerospike's Telnet Port
Aerospike also provides a telnet service which is typically configured to
port 3003. This service provides the same functionality as **asinfo**, once
connected type the same commands you would normally pass to **asinfo's** **-v**
option.

{{#info}}
**Note:** This option is not available for [security](/docs/guide/security) enabled servers.
This option can also be disabled by commenting out or removing the info stanza from aerospike.conf. 
It is also possible to bind this service to localhost by setting the address field to 127.0.0.1.
{{/info}}

**Example:**

```
$ telnet 127.0.0.1 3003
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
namespaces
user_profile;test;bar
```

### Commands:

For a comprehensive list of commands see [Info Command Reference](/docs/reference/info)
