---
title: Nagios Integrations Guide
---

The Aerospike Nagios plugin is available on [Github](https://github.com/aerospike/aerospike-nagios).

The aerospike\_nagios.py script simplifies nagios configurations for Aerospike clusters.
The goal is to reduce the complexity to 2 simple steps.

1. Copy aerospike\_nagios.py and dependencies to your Nagios server
2. Add aerospike configs into Nagios

### Features

- Can monitor any stat returned by
  - `$ asinfo -v 'statistics' [-h host]`
  - `$ asinfo -v 'namespace/<NAMESPACE NAME>' [-h host]`
  - `$ asinfo -v 'dc/<DATACENTER>' [-h host]`
  - `$ asinfo -v 'latency:hist=<LATCENCY STAT>' [-h host]`

### Requirements
Additional python modules are required and installed using pip:
```
sudo pip install -r requirements.txt
```

### Getting Started

1. Copy aerospike\_nagios.py to your prefered scripts dir

    ` eg: /opt/aerospike/bin/`

2. Copy aerospike\_schema.yaml and ssl directory to the same directory

3. Copy examples/aerospike.cfg into your nagios conf.d directory

   `/etc/nagios/conf.d if installed from repo`
   `/usr/local/nagios/etc/objects if installed from source`

4. Edit aerospike.cfg to add your aerospike hosts into the hostgroup

5. Restart/reload nagios

### Aerospike nagios Plugin

See *aerospike\_nagios.py*, this is the file that nagios will schedule to perform
queries against Aerospike. Other than copying it to the appropriate location,
you are not required to interact with it.

###  Usage

```bash
$ python /opt/aerospike/bin/aerospike_nagios.py --help
usage: aerospike_nagios.py [-u] [-U USER] [-P [PASSWORD]]
                           [--credentials-file CREDENTIALS]
                           [--auth-mode AUTH_MODE] [-v]
                           [-n NAMESPACE | -l LATENCY | -x DC] -s STAT
                           [-p PORT] [-h HOST] -c CRIT -w WARN
                           [--timeout TIMEOUT] [--tls-enable]
                           [--tls-name TLS_NAME] [--tls-keyfile TLS_KEYFILE]
                           [--tls-keyfile-pw TLS_KEYFILE_PW]
                           [--tls-certfile TLS_CERTFILE]
                           [--tls-cafile TLS_CAFILE] [--tls-capath TLS_CAPATH]
                           [--tls-ciphers TLS_CIPHERS]
                           [--tls-protocols TLS_PROTOCOLS]
                           [--tls-cert-blacklist TLS_CERT_BLACKLIST]
                           [--tls-crl-check] [--tls-crl-check-all]

optional arguments:
  -u, --usage, --help   Show this help message and exit
  -U USER, --user USER  user name
  -P [PASSWORD], --password [PASSWORD]
                        password
  --credentials-file CREDENTIALS
                        Path to the credentials file. Use this in place of
                        --user and --password.
  --auth-mode AUTH_MODE
                        Authentication mode. Values: ['EXTERNAL_INSECURE',
                        'INTERNAL', 'EXTERNAL'] (default: INTERNAL)
  -v, --verbose         Enable verbose logging
  -n NAMESPACE, --namespace NAMESPACE
                        Namespace name. eg: bar
  -l LATENCY, --latency LATENCY
                        Options: see output of asinfo -v 'latency:hist' -l
  -x DC, --xdr DC       Datacenter name. eg: myDC1
  -s STAT, --stat STAT  Statistic name. eg: cluster_size
  -p PORT, ---port PORT
                        PORT for Aerospike server (default: 3000)
  -h HOST, --host HOST  HOST for Aerospike server (default: 127.0.0.1)
  -c CRIT, --critical CRIT
                        Critical level
  -w WARN, --warning WARN
                        Warning level
  --timeout TIMEOUT     Set timeout value in seconds to node level operations.
                        TLS connection does not support timeout. (default: 5)
  --tls-enable          Enable TLS
  --tls-name TLS_NAME   The expected name on the server side certificate
  --tls-keyfile TLS_KEYFILE
                        The private keyfile for your client TLS Cert
  --tls-keyfile-pw TLS_KEYFILE_PW
                        Password to load protected --tls-keyfile
  --tls-certfile TLS_CERTFILE
                        The client TLS cert
  --tls-cafile TLS_CAFILE
                        The CA for the server's certificate
  --tls-capath TLS_CAPATH
                        The path to a directory containing CA certs and/or
                        CRLs
  --tls-ciphers TLS_CIPHERS
                        Ciphers to include. See https://www.openssl.org/docs/m
                        an1.1.0/man1/ciphers.html for cipher list format
  --tls-protocols TLS_PROTOCOLS
                        The TLS protocol to use. Available choices: TLSv1,
                        TLSv1.1, TLSv1.2, all. An optional + or - can be
                        appended before the protocol to indicate specific
                        inclusion or exclusion.
  --tls-cert-blacklist TLS_CERT_BLACKLIST
                        Blacklist including serial number of certs to revoke
  --tls-crl-check       Checks SSL/TLS certs against vendor's Certificate
                        Revocation Lists for revoked certificates. CRLs are
                        found in path specified by --tls-capath. Checks the
                        leaf certificates only
  --tls-crl-check-all   Check on all entries within the CRL chain
```
```
	 -U user (Enterprise only)
	 -P password (Enterprise only)
```

### Examples
To monitor a specific general statistic:  
```
aerospike_nagios.py -h YOUR_ASD_HOST -s STAT_NAME -w WARN_LEVEL -c CRIT_LEVEL
```

To monitor a specific statistic in a namepsace:  
```
aerospike_nagios.py -h YOUR_ASD_HOST -s STAT_NAME -n YOUR_NAMESPACE -w WARN_LEVEL -c CRIT_LEVEL
```

To monitor a specfic statistic in xdr:
```
aerospike_nagios.py -h YOUR_ASD_HOST -s STAT_NAME -x DATACENTER -w WARN_LEVEL -c CRIT_LEVEL
```

To monitor latency statistics (pre-3.9):
```
aerospike_nagios.py -h YOUR_ASD_HOST -s <1ms|8ms|64ms>  -l <reads|writes|writes_reply|proxy> -w WARN_LEVEL -c CRIT_LEVEL
```

To monitor latency statistics (ASD 3.9+):
```
aerospike_nagios.py -h YOUR_ASD_HOST -s <1ms|8ms|64ms>  -l {NAMESPACE}-<read|write|proxy|udf> -w WARN_LEVEL -c CRIT_LEVEL
```
eg: `aerospike_nagios.py -h localhost -s 1ms  -l {test}-read -w 8 -c 10`

To utilize SSL/TLS standard auth:
```
aerospike_nagios.py -h YOUR_ASD_HOST -p YOUR_SECURED_PORT -s STAT_NAME --tls-enable --tls-cafile YOUR_CA_PEM --tls-name YOUR_ASD_CERT_NAME -w WARN_LEVEL -c CRIT_LEVEL
```

### Alert Levels

Warning and Critical thresholds are specified according to [Nagios' format](https://nagios-plugins.org/doc/guidelines.html#THRESHOLDFORMAT).

To not use warning and/or critical levels, set them to 0.

Example usage can be found in the examples/aerospike.cfg file. 

### Authentication

You can specify User and Password for authentication via the -U/--user and -P/--password parameters.
The Password is also an interactive prompt if you leave it empty.

If this is not preferable, you can also specify a credentials file with -c/--credentials-file. 
It is a simple 2 line file, with the username and password on each line, in that order. 
With this method, the credentials file can be secured via other means (eg: chmod 600) and prevent snooping.

`AuthMode` is optional parameter to specify authentication mode. It's default value is INTERNAL.
