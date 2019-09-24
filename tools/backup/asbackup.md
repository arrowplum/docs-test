---
title: Aerospike Backup (asbackup)
description: Learn how to operate the Aerospike backup function, including command options and estimating space required for backup files.

breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: Backup and Restore
    url: /docs/v3/Backup and Restore.html
---


The `asbackup` utility is used to backup namespaces or sets from an Aerospike cluster to local storage. `asbackup` concurrently queries multiple nodes in the cluster. By default, up to 10 nodes are backed up simultaneously. This limit can be changed with the `--parallel` command line option. Data from each cluster node, is backed up on a partition-by-partition basis. If a record is created or updated after its partition was backed up, the backup will not reflect that.

{{#note}}While a cluster is migrating (i.e., rebalancing due to a node failure), any taken backups may not be fully consistent. There may be multiple copies of the same record or missing records. Use the `--no-cluster-change` option to make `asbackup` abort, in case it detects pending migrations.{{/note}}

When run with the `--directory` option, `asbackup` creates multiple `.asb` backup files in the given directory. The backup consists of all created files. Alternatively, `--output-file` makes `asbackup` store the complete backup in the given single file. If `-` is specified as the file, `asbackup` writes the backup to `stdout`. This allows for pipelines:

```bash
asbackup --output-file - [...] | gzip -1 [...] >backup.asb.gz
```
## Usage

The `-Z` or `--usage` option of `asbackup` gives an overview of all supported command line options.

```bash
$ asbackup --usage
```

The most basic form of running `asbackup` is to just specify the cluster to backup (`--host`), the namespace to backup (`--namespace`), as well as the local directory for the backup files (`--directory`). Suppose that we have a cluster that contains a node with IP address `1.2.3.4`. To backup the `test` namespace on this cluster to the directory `backup_2015_08_24`, we would issue the following command.

```bash
$ asbackup --host 1.2.3.4 --namespace test --directory backup_2015_08_24
```

## Estimating the Backup Size

When passing the `--estimate` command line option to `asbackup` (and skipping `--directory` and `--output-file`), `asbackup` creates a temporary test backup of 10,000 records from the namespace. It then outputs, based on the observed record sizes, an estimate of the average size of a record in the backup. In order to estimate the total size of the backup file or files, multiply this size by the number of records in the namespace and add 10% for indexes and overhead.

## Incremental Backup

Beginning from Server Version 3.12, timestamps can be specified so only records updated since timestamp X are backed up. An operational routine can be established to do incremental daily backups. Please see `--modified-after` option in Data Options section.

## Connection Options

| Option | Default | Description|
|--------|---------|------------|
| `-h <host1>[:<tlsname1>][:<port1>][,...]` or `--host <host1>[:<tlsname1>][:<port1>][,...]` | 127.0.0.1 | The host that acts as the entry point to the cluster. Any of the cluster nodes can be specified. The remaining cluster nodes will be automatically discovered.|
| `-p <port>` or `--port <port>` | 3000 | Port to connect to. |
| `-U <user>` or `--user <user>` | - | User name with read permission. **Mandatory if the server has security enabled.** |
| `-P<password>` or `--password`| - | Password to authenticate the given user. The first form passes the password on the command line. The second form prompts for the password. |
| `-l <host1>[:<tlsname1>]:<port1>[,...]` or `--node-list <addr1>[:<tlsname1>]:<port1>[,...]` | All nodes | While `--host`/`--port` make `asbackup` automatically discover and backup all cluster nodes, `--node-list` can be used to backup only a subset of the cluster nodes. |
| `-w <nodes>` or `--parallel <nodes>` | 10 | Maximal number of nodes to backup in parallel. Backup will concurrently scan up to the requested number of nodes. **Setting this number too high may result in client overload.** |
| `--tls-enable` | disabled | Indicates a TLS connection should be used. |
| `--service-alternate` | false | Use to connect to [`alternate-access-address`](/docs/reference/configuration/#alternate-access-address) when the cluster nodes publish IP addresses through [`access-address`](/docs/reference/configuration/#access-address) which are not accessible over WAN and alternate IP addresses accessible over WAN through [`alternate-access-address`](/docs/reference/configuration/#alternate-access-address). |


The following example creates a backup of cluster nodes `1.2.3.4` and `5.6.7.8` using the default Aerospike port of `3000`.

```bash
$ asbackup --node-list 1.2.3.4:3000,5.6.7.8:3000 --namespace test --directory backup_2015_08_24
```

## TLS Options

| Option | Default | Description|
|--------|---------|------------|
| `--tls-cafile=TLS_CAFILE` | | Path to a trusted CA certificate file. |
| `--tls-capath=TLS_CAPATH` | | Path to a directory of trusted CA certificates. |
| `--tls-protocols=TLS_PROTOCOLS` | | Set the TLS protocol selection criteria. This format is the same as Apache's SSLProtocol documented at https://httpd.apache.org/docs/current/mod/mod_ssl.html#sslprotocol . If not specified the asbackup will use '-all +TLSv1.2' if has support for TLSv1.2, otherwise it will be '-all +TLSv1'. |
| `--tls-cipher-suite=TLS_CIPHER_SUITE` | | Set the TLS cipher selection criteria. The format is the same as Open_sSL's Cipher List Format documented at https://www.openssl.org/docs/manmaster/man1/ciphers.html . |
| `--tls-keyfile=TLS_KEYFILE` | | Path to the key for mutual authentication (if Aerospike Cluster is supporting it). |
| `--tls-keyfile-password=TLS_KEYFILE_PASSWORD` | | Password to load protected tls-keyfile. It can be one of the following:<br/>1) Environment varaible: 'env:&lt;VAR&gt;'<br/>2) File: 'file:&lt;PATH&gt;'<br/>3) String: 'PASSWORD'<br/>User will be prompted on command line if --tls-keyfile-password specified and no password is given. |
| `--tls-certfile=TLS_CERTFILE  <path>` | | Path to the chain file for mutual authentication (if Aerospike Cluster is supporting it). |
| `--tls-cert-blacklist <path>` | | Path to a certificate blacklist file. The file should contain one line for each blacklisted certificate. Each line starts with the certificate serial number expressed in hex. Each entry may optionally specify the issuer name of the certificate (serial numbers are only required to be unique per issuer). Example: 867EC87482B2 /C=US/ST=CA/O=Acme/OU=Engineering/CN=TestChainCA |
| `--tls-crl-check` | | Enable CRL checking for leaf certificate. An error occurs if a valid CRL files cannot be found in tls_capath. |
| `--tls-crl-checkall` | | Enable CRL checking for entire certificate chain. An error occurs if a valid CRL files cannot be found in tls_capath. |

The tlsname is only used when connecting with a secure TLS enabled server.
The following example creates a backup of cluster nodes `1.2.3.4` and `5.6.7.8` using the default Aerospike port of `3000` with tls configured.  
HOST is "&lt;host1&gt;[:&lt;tlsname1&gt;][:&lt;port1&gt;],...".

```bash
asbackup --host 1.2.3.4:cert1:3000,5.6.7.8:cert2:3000 --namespace test --directory backup_2015_08_24 --tls-enable --tls-cafile /cluster_name.pem --tls-protocols TLSv1.2 --tls-keyfile /cluster_name.key --tls-certfile /cluster_name.pem
``` 

## Output Options

| Option | Default | Description|
|--------|---------|------------|
| `-d <path>` or `--directory <path>` | - | Directory to store the `.asb` backup files in. If the directory does not exist, it will be created before use. **Mandatory, unless `--output-file` or `--estimate` is given.** |
| `-o <path>` or `--output-file <path>` | - | The single file to write the backup to. `-` means `stdout`. **Mandatory, unless `--directory` or `--estimate` is given.** |
| `-e` or `--estimate` | - | Specified in lieu of `--directory` or `--output-file`, estimates the average size of a single record in the backup file. Useful for estimating the expected size of a backup before actually starting it. Multiply the returned value by the number of records in the namespace and add 10% for overhead. |
| `-F <limit>` or `--file-limit <limit>` | 250 MiB | File size limit (in MiB) for `--directory`. If a `.asb` backup file crosses this size threshold, `asbackup` will switch to a new file. |
| `-r` or `--remove-files` | - | Clear directory or remove output file. By default, `asbackup` refuses to write to a non-empty directory or to overwrite an existing backup file. This option clears the given `--directory` or removes an existing `--output-file`. Not allowed in configuration file.|
| `-C` or `--compact` | - | Do not base-64 encode BLOB values. For better readability of backup files, `asbackup` base-64 encodes BLOB values by default. This option disables the encoding step, which saves space in the backup file. However, be prepared to encounter odd-looking binary data in your backup files. |
| `-N <bandwidth>` or `--nice <bandwidth>` | - | Throttles `asbackup`'s write operations to the backup file(s) to not exceed the given bandwidth in MiB/s. Effectively also throttles the scan on the server side as `asbackup` refuses to accept more data than it can write. |

## Data Selection Options

| Option | Default | Description|
|--------|---------|------------|
| `-n <namespace>` or `--namespace <namespace>` | - | Namespace to backup. **Mandatory.** |
| `-s <set>` or `--set <set>` | All sets | The set to backup. |
| `-B  <bin1>[,<bin2>[,...]]` or `--bin-list <bin1>[,<bin2>[,...]]` | All bins | The bins to backup. |
| `-x` or `--no-bins` | - | Only backup record metadata (digest, TTL, generation count, key). **WARNING: No data (bin contents) is backed up. Also, this is unrelated to the single-bin option in the Aerospike server configuration file.** |
| `-R` or `--no-records` | - | Do not backup any record data (metadata or bin data). By default, `asbackup` includes record data, secondary index definitions, and UDF modules. |
| `-I` or `--no-indexes` | - | Do not backup any secondary index definitions. See above. |
| `-u` or `--no-udfs` | - | Do not backup any UDF modules. See above. |
| `-% <percentage>` or `--percent <percentage>` | 100% | Fraction of cluster records to backup. This option is useful for sampling, e.g., to backup a randomly selected 1% of the total cluster data to in order to run analytics on the sample. |
| `-a <YYYY-MM-DD_HH:MM:SS>` or `--modified-after <YYYY-MM-DD_HH:MM:SS>` | - | Backup data with last-update-time after the specified date-time. The system's local timezone applies. Available in Server 3.12 and above.  |
| `-b <YYYY-MM-DD_HH:MM:SS>` or `--modified-before <YYYY-MM-DD_HH:MM:SS>` | - | Backup data with last-update-time before the specified date-time. The system's local timezone applies. Available in Server 3.12 and above.  |

## Configuration File Options

asbackup can be configured by using tools configuration files. Please see [Aerospike Tools Configuration](/docs/tools/conffile) for more details. The following options effect configuration file behavior.

| Option | Default | Description|
|--------|---------|------------|
| `--no-config-file ` | disabled | Do not read any configuration file. The configuration files options `--no-config-file` and `only-config-file` are mutually exclusive. |
| `--instance <name>` | - | Section with these instance is read. e.g in case instance `a` is specified sections cluster_a, asbackup_a is read. |
| `--config-file <path>` | - | Read this file after default configuration file. |
| `--only-config-file <path>` | - | Read only this configuration file. The configuration files options `--no-config-file` and `only-config-file` are mutually exclusive. |

## Other Options

| Option | Default | Description|
|--------|---------|------------|
| `-f <priority>` or `--priority <priority>` | 0 | The level of scan concurrency on each individual node. Higher values mean faster backups. **For higher priorities, do monitor the read/write performance of your application to ensure that cluster performance is not affected too much. Allowed values: 0 (auto), 1 (low), 2 (medium), 3 (high). |
| `-c` or `--no-cluster-change` | - | Terminate in case of pending migrations. When the cluster configuration changes (e.g., when a node fails), data starts to migrate between the cluster nodes to rebalance the cluster. This may lead to backups that are not fully consistent (duplicate records, missing records). To guarantee a good, complete backup, use this option. |
| `-v` or `--verbose` | disabled | Output considerably more information about the running backup. |

