---
title: Backup and Restore (asbackup/asrestore)
description: Learn how backup and restore functions operate, what data is backed up, how backup files are created and how data is restored.

breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
---

Aerospike provides the ability to backup and restore your cluster data. Under normal circumstances, data replication (within a cluster) and cross data center replication (XDR) ensure that data is not lost even when there are hardware failures or network outages. However, it is good practice to periodically create backups for easier recovery from catastrophic data center failures or administrative accidents. 

The backup/restore utilities available are:

- [asbackup](/docs/tools/backup/asbackup.html) - Backup all data from a namespace or a specific set in the namespace using Aerospike's standard client scan API.
- [asrestore](/docs/tools/backup/asrestore.html) - Restore namespace or set data from a backup created by `asbackup` using Aerospike's standard client API.

The backup/restore utilities are part of the ***aerospike-tools*** installation package, and is also source code available on [github](https://github.com/aerospike/aerospike-tools-backup).

### What Data is Backed Up

You have the option of backing up:

- Records
  - Record metadata (digest, TTL, generation count, key)
  - Regular bins (string, integer, binary)
  - Complex data type (CDT) bins (list, map)
- Secondary index definitions
- User defined function (UDF) modules

### Incremental Backup

Starting from Server and Tools version 3.12, timestamp can be specified when initiating a backup to indicate the time-period of interest. Only records whose last-update-time satisfies the specified criteria (before time T1 and/or after time T2) will be backed up.

### How the Backup Files are Created

Backup files are human-readable text files. For details, see the [File Format Specification](https://github.com/aerospike/aerospike-tools-backup/tree/master#backup-file-format) in the source code repository.

The most basic form of running `asbackup` is to just specify the cluster to backup (`--host`), the namespace to backup (`--namespace`), as well as the local directory for the backup files (`--directory`). Suppose that we have a cluster that contains a node with IP address `1.2.3.4`. To backup the `test` namespace on this cluster to the directory `backup_2015_08_24`, we would issue the following command.

```bash
$ asbackup --host 1.2.3.4 --namespace test --directory backup_2015_08_24
```

The backup program `asbackup` reads records from the cluster to be backed up and stores them in a set of backup files under the directory specified with the `--directory` command line option. By default, each backup file is limited to 250 MiB. When this limit is crossed, `asbackup` starts a new file. Alternatively, the cluster can be backed up to a single file or to `stdout` using `--output-file` instead of `--directory`.

The `--no-cluster-change` option aborts the backup, if a cluster node fails during the backup. Node failures cause the cluster to rebalance and, thus, data migration between nodes. In case the backup is aborted, please restart it after data migration is complete.

If the backup is aborted, the backup files created up to that point will remain around, and by default, `asbackup` will refuse to overwrite existing backup files. When restarting the backup, you may either backup to a different directory or file, or use `--remove-files` to have `asbackup` remove the existing backup file or files.

### How the Data is Restored

Backups are cluster-agnostic. The size and configuration of the cluster from which the backup was taken can be completely different from the size and configuration of the cluster to which the data is restored. Data backed up from a 5-node cluster, for example, can be restored to a 4-node or 7-node cluster. The restore process will always evenly distribute the data across the cluster nodes.

The most basic form of running `asrestore` is to just specify the cluster to restore (`--host`) and the local directory containing the backup files (`--directory`). Suppose that we have a cluster that contains a node with IP address `1.2.3.4`. To restore a backup from directory `backup_2015_08_24`, we would issue the following command.

```bash
$ asrestore --host 1.2.3.4 --directory backup_2015_08_24
```

By default, the backup is restored to the namespace that it was taken from. The `--namespace` option can be used to restore to a different namespace. Suppose that the above backup was taken from namespace `test` and we would like to restore it to namespace `prod`. We would then issue the following command.

```bash
$ asrestore --host 1.2.3.4 --directory backup_2015_08_24 --namespace test,prod
```

`asrestore` reads backup files from the directory specified with the `--directory` option. Alternatively, if the backup consists of a single file created with the `--output-file` option of `asbackup`, the `--input-file` option can be used to make `asrestore` read that single file or `stdin`.

Restored records preserve the record's original TTL. Last-update-time and generation will be new.

### The Write Policy

The read data is restored to a namespace in a cluster using the standard client API for storing records. The namespace may already contain existing records and, by default, the write policy works as follows.

- If the record from the backup is expired (based on its TTL value), it is ignored.
- If the record does not exist in the namespace, it is restored from the backup.
- If a newer version of the record (higher or same generation count) exists in the namespace, the record from the backup is ignored.
- If an older version of the record (lower generation count) exists in the namespace, the record is restored from the backup. If the record in the namespace contains bins that are not present in the backup, those bins are preserved.

The following `asrestore` command line options modify this write policy.

- `--unique` - Do not touch any existing records, regardless of generation counts.
- `--no-generation` - Overwrite any existing records, regardless of generation counts.
- `--replace` - When restoring a record from the backup, do not preserve bins that are not present in the backup.

{{#note}}`asrestore` will only restore backups that are *Version 3.0* or higher. To restore a backups from *Version 2.0 to Version 3.0* please contact **Aerospike Support**.{{/note}}
