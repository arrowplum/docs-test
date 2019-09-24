<a name="validate"></a>
### Validate Aerospike Tools Installation

{{#steps}}
{{#steps-step 5 "Verify the Aerospike Tools package installation on the system" markdown=true}}

On a Mac machine use the following command to verify that the aerospike-tools package is installed

```bash
pkgutil --pkgs | grep 'aerospike.tools'
```

---
The above command should output the tools package installed
Sample:
```bash
$ pkgutil --pkgs | grep 'aerospike.tools'
com.aerospike.tools.3.13.0.1
``` 
---
{{/steps-step}}

{{#steps-step 6 "Run each of the Aerospike tools to validate" markdown=true}}

---
asadm: Aerospike **[Admin](/docs/tools/asadm)** is an interactive python utility primarily used to get summary info for the current health of a cluster and also for executing dynamic configuration and tuning commands across the cluster. Provide the user name and password if security is enabled on the cluster.
```bash
$ asadm
Aerospike Interactive Shell, version 0.1.11

Found 1 nodes
Online:  10.0.0.102:3000

Admin> 

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
2017-08-31 22:15:09 UTC [INF] [52989] Starting 100% backup of 127.0.0.1 (namespace: test, set: [all], bins: [all], after: [none], before: [none]) to /Users/veera/output
2017-08-31 22:15:09 UTC [INF] [52989] [src/main/aerospike/as_cluster.c:96][as_cluster_add_nodes_copy] Add node BB99E8005270008 127.0.0.1:3000
2017-08-31 22:15:09 UTC [INF] [52989] Processing 1 node(s)
2017-08-31 22:15:09 UTC [INF] [52989] Node ID             Objects        Replication    
2017-08-31 22:15:09 UTC [INF] [52989] BB99E8005270008     0              1              
2017-08-31 22:15:09 UTC [INF] [52989] Namespace contains 0 record(s)
2017-08-31 22:15:09 UTC [INF] [52989] Created new backup file /Users/veera/output
2017-08-31 22:15:09 UTC [INF] [53013] Starting backup for node BB99E8005270008
2017-08-31 22:15:09 UTC [INF] [53013] No secondary indexes
2017-08-31 22:15:09 UTC [INF] [53013] Backing up 0 UDF file(s)
2017-08-31 22:15:09 UTC [INF] [53013] Completed backup for node BB99E8005270008, records: 0, size: 42 (~0 B/rec)
2017-08-31 22:15:10 UTC [INF] [53012] Backed up 0 record(s), 0 secondary index(es), 0 UDF file(s) from 1 node(s), 42 byte(s) in total (~0 B/rec)
vagrant@vagrant-ubuntu-trusty-64:~$ 

```
---
{{/steps-step}}
{{/steps}}

