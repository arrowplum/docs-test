---
title: Aerospike Restore (asrestore)
introduction: Learn how to restore data saved by an Aerospike backup (asbackup), including command options.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: Backup and Restore
    url: /docs/v3/Backup and Restore.html
---

The `asrestore` utility restores backups created with [asbackup](/docs/tools/backup/asbackup.html). If the namespace on the cluster already contains existing records, a configurable write policy determines, which records take precedence - records in the namespace or records from the backup. The [Write Policy](/docs/tools/backup/index.html#the-write-policy) section on the overview page has more details on this.

If the initial insertion of a record fails `asrestore` will retry inserting the record 10 times.  Between tries there is a 1 second pause and the error is written to the logs at the debug severity level.  If a record is not written after 10 tries, the restore is aborted with the error message "Too many errors, giving up".  The specific errors in which a retry is not effected are "record exists" (when the --unique option is used), "generation mismatch" (unless if the --no-generation option is used) and "invalid username or password".

When run with the `--directory` option, `asrestore` expects multiple `.asb` backup files in the given directory. Alternatively, `--input-file` makes `asrestore` read the complete backup from the given single file. If `-` is specified as the file, `asrestore` reads the backup from `stdin`. This allows for pipelines:

```bash
cat backup.asb.gz | gunzip | asrestore --input-file - [...]
```

# Usage

The `-Z` or `--usage` option of `asrestore` gives an overview of all supported command line options.

```bash
$ asrestore --usage
```

The most basic form of running `asrestore` is to just specify the cluster to restore (`--host`) and the local directory containing the backup files (`--directory`). Suppose that we have a cluster that contains a node with IP address `1.2.3.4`. To restore a backup from directory `backup_2015_08_24`, we would issue the following command.

```bash
$ asrestore --host 1.2.3.4 --directory backup_2015_08_24
```

By default, the backup is restored to the namespace that it was taken from. The `--namespace` option can be used to restore to a different namespace. Suppose that the above backup was taken from namespace `test` and we would like to restore it to namespace `prod`. We would then issue the following command.

```bash
$ asrestore --host 1.2.3.4 --directory backup_2015_08_24 --namespace test,prod
```

## Connection Options

| Option | Default | Description|
|--------|---------|------------|
| `-h <host>` or `--host <host>` | 127.0.0.1 | The host that acts as the entry point to the cluster. Any of the cluster nodes can be specified. The remaining cluster nodes will be automatically discovered. |
| `-p <port>` or `--port <port>` | 3000 | Port to connect to. |
| `-U <user>` or `--user <user>` | - | User name with write permission. **Mandatory if the server has security enabled.** |
| `-P<password>` or `--password`| - | Password to authenticate the given user. The first form passes the password on the command line. The second form prompts for the password. |
| `-t <threads>` or `--threads <threads>` | 20 | The number of client threads to spawn for writing to the cluster. Higher numbers mean faster restores, which may, however, have a negative impact on server performance. |
| `--tls-enable` | disabled | Indicates a TLS connection should be used. |
| `-T <timeout>` or `--timeout <timeout>` | 10000 | Timeout (ms) for Aerospike commands to write records, create indexes and create UDFs. |

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
The following example restores a cluster backup to node `1.2.3.4` using the default Aerospike port of `3000` with tls configured.  
HOST is "&lt;host&gt;[:&lt;tlsname&gt;][:&lt;port&gt;]".

```bash
asbackup --host 1.2.3.4:cert1:3000 --directory backup_2015_08_24 --namespace test --tls-enable --tls-cafile /cluster_name.pem --tls-protocols TLSv1.2 --tls-keyfile /cluster_name.key --tls-certfile /cluster_name.pem 
```

## Input Options

| Option | Default | Description|
|--------|---------|------------|
| `-d <path>` or `--directory <path>` | - | Directory from which to read the `.asb` backup files. **Mandatory, unless `--input-file` is given.** |
| `-i <path>` or `--input-file <path>` | - | The single file from which to read the backup. `-` means `stdin`. **Mandatory, unless `--directory` is given.** |
| `-N <bandwidth>,<TPS>` or `--nice <bandwidth>,<TPS>` | - | Throttles `asrestore`'s read operations from the backup file(s) to not exceed the given I/O bandwidth in MiB/s and its database write operations to not exceed the given number of transactions per second. Useful to limit the impact of `asrestore` on server performance. |

## Data Selection Options

| Option | Default | Description|
|--------|---------|------------|
| `-n <original-ns>[,<new-ns>]` or `--namespace <original-ns>[,<new-ns>]` | Original namespace | Namespace to be restored. By default, `asrestore` restores a backup to the namespace from which it was taken. If this option is specified and the namespace from which the backup was taken does not match `<original-ns>`, `asrestore` aborts with an error. This ensures that we restore the data that we intend to restore. If `<new-ns>` is specified, the backup will be restored to `<new-ns>` instead of the namespace from which it was taken. |
| `-s <set1>[,<set2>[,...]]` or `--set-list <set1>[,<set2>[,...]]` | All sets | The sets to restore. |
| `-B <bin1>[,<bin2>[,...]]` or `--bin-list <bin1>[,<bin2>[,...]]` | All bins | The bins to restore. |
| `-R` or `--no-records` | - | Do not restore any record data (metadata or bin data). By default, `asrestore` restores record data, secondary index definitions, and UDF modules. |
| `-I` or `--no-indexes` | - | Do not restore any secondary index definitions. See above. |
| `-F` or `--no-udfs` | - | Do not restore any UDF modules. See above. |

## Write Policy Options

| Option | Default | Description|
|--------|---------|------------|
| `-u` or `--unique` | - | Existing records take precedence. With this option, only records that do not exist in the namespace are restored, regardless of generation numbers. *If a record exists in the namespace, the record from the backup is ignored.* |
| `-g` or `--no-generation` | - | Records from backups take precedence. This option disables the generation check. With this option, records from the backup always overwrite records that already exist in the namespace, regardless of generation numbers. {{#warn}}By using this option you may lose a more recent version of your data by overwriting it with an older version.{{/warn}} |
| `-r` or `--replace` | - | Replace records. This controls how records from the backup overwrite existing records in the namespace. By default, restoring a record from a backup only replaces the bins contained in the backup; all other bins of an existing record remain untouched. With this option, the existing record is completely replaced; i.e., any bins that that are not contained in the backup are discarded. |

## Configuration File Options

asrestore can be configured by using tools configuration files. Please see [Aerospike Tools Configuration](/docs/tools/conffile) for more details. The following options effect configuration file behavior.

| Option | Default | Description|
|--------|---------|------------|
| `--no-config-file ` | disabled | Do not read any configuration file. |
| `--instance <name>` | - | Section with these instance is read. e.g in case instance `a` is specified sections cluster_a, asrestore_a is read. |
| `--config-file <path>` | - | Read this file after default configuration file. |
| `--only-config-file <path>` | - | Read only this configuration file. |

## Other Options

| Option | Default | Description|
|--------|---------|------------|
| `-v` or `--verbose` | disabled | Output considerably more information about the running restore. |
| `-L` or `--indexes-last` | - | Create indexes after restoring everything else. By default, indexes are restored before everything else, which can prevent costly SSD reads required to build the indexes. |
| `-w` or `--wait` | - | Wait for secondary indexes to finish building before proceeding. Wait for restored UDFs to be distributed across the cluster. |


{{#info}}If would like to change the target set to which to restore data, please contact **Aerospike Support** for assistance.{{/info}}
