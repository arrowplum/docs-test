<a name="validate"></a>
### Validate Aerospike Tools Installation

{{#steps}}
{{#steps-step 4 "Verify the Aerospike Tools package installation on the system" markdown=true}}

On a Debian machine use the following command to verify that the aerospike-tools package is installed

```bash
dpkg-query -l | grep aerospike-tools
```

---
The above command should output a line with starting letters as "ii". The first letter 'i' indicates the desired package state as install and the second letter 'i' indicates the current package state also as install.

Sample:
```bash
$ dpkg-query -l | grep aerospike-tools
ii  aerospike-tools                3.13.0.1                    amd64        Aerospike server tools.
``` 
---
{{/steps-step}}

{{#steps-step 5 "Run each of the Aerospike tools to validate" markdown=true}}

---
asadm: Aerospike **[Admin](/docs/tools/asadm)** is an interactive python utility primarily used to get summary info for the current health of a cluster and also for executing dynamic configuration and tuning commands across the cluster. Provide the user name and password if security is enabled on the cluster.
```bash
$ asadm
Aerospike Interactive Shell, version 0.1.11

Found 1 nodes
Online:  10.0.2.15:3000

```
aql: The **[aql](/docs/tools/aql)** tool provides an SQL-like command line interface for database, UDF and index management. Provide the user name and password if security is enabled on the cluster.
```bash
$ aql
Aerospike Query Client
Version 3.13.0.1
C Client Version 4.1.6
Copyright 2012-2016 Aerospike. All rights reserved.
aql> 

```
asinfo: **[asinfo](/docs/tools/asinfo)** is a command-line utility that provides an interface to Aerospike's cluster command and control functions. This includes the ability to change server configuration parameters while the Aerospike is running. Provide the user name and password if security is enabled on the cluster.
```bash
$ asinfo -v edition
Aerospike Enterprise Edition

```
asbackup: The **[asbackup](https://www.aerospike.com/docs/tools/backup/asbackup.html)** utility is used to backup namespaces or sets from an Aerospike cluster to local storage. Provide the user name and password if security is enabled on the cluster.
```bash
$ asbackup -n test -o ~/output
2017-09-01 01:18:12 GMT [INF] [ 2483] Starting 100% backup of 127.0.0.1 (namespace: test, set: [all], bins: [all], after: [none], before: [none]) to /home/vagrant/output
2017-09-01 01:18:12 GMT [INF] [ 2483] [src/main/aerospike/as_cluster.c:96][as_cluster_add_nodes_copy] Add node BB95BC488270008 127.0.0.1:3000
2017-09-01 01:18:12 GMT [INF] [ 2483] Processing 1 node(s)
2017-09-01 01:18:12 GMT [INF] [ 2483] Node ID             Objects        Replication    
2017-09-01 01:18:12 GMT [INF] [ 2483] BB95BC488270008     0              1              
2017-09-01 01:18:12 GMT [INF] [ 2483] Namespace contains 0 record(s)
2017-09-01 01:18:12 GMT [INF] [ 2483] Created new backup file /home/vagrant/output
2017-09-01 01:18:12 GMT [INF] [ 2502] Starting backup for node BB95BC488270008
2017-09-01 01:18:12 GMT [INF] [ 2502] No secondary indexes
2017-09-01 01:18:12 GMT [INF] [ 2502] Backing up 0 UDF file(s)
2017-09-01 01:18:12 GMT [INF] [ 2502] Completed backup for node BB95BC488270008, records: 0, size: 42 (~0 B/rec)
2017-09-01 01:18:13 GMT [INF] [ 2501] Backed up 0 record(s), 0 secondary index(es), 0 UDF file(s) from 1 node(s), 42 byte(s) in total (~0 B/rec)
```
---
{{/steps-step}}
{{/steps}}

