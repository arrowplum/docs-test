---
title: Overview
description: Discover the handy tools that give visibility, validate, help set and get validate values and more from the Aerospike cluster. 
---

Aerospike contains a number of tools. These are usually, but not always, run on
the server. Some tools are run from other nodes - like maintenance machines,
which manage backups. 

Tools can be configured by using tools configuration files. 
Please see **[Aerospike Tools Configuration](/docs/tools/conffile)** for details.

#### [AQL](/docs/tools/aql) - aql
This SQL-like command line can be used for basic validation and testing of most
database functionality, as well as managing indexes and UDFs.

#### [Backup and Restore](/docs/tools/backup) - asbackup/asrestore
Use a node outside the cluster and, in a distributed way, pull out all the
cluster's data into a text file. Or, restore the data from one of these files.
Source is included, allowing this tool to be modified.

#### [Aerospike Admin](/docs/tools/asadm) - asadm
This command line tool gives an immediate view into the cluster - its size, and 
its health. This tool can work with running cluster (Cluster mode) as well as 
with collectinfo files (Collectinfo-analyser mode) and with aerospike log files (Log-analyser mode).
Also in Cluster mode, it provides functions to execute Aerospike commands across the entire cluster. 

#### [Log Latency Tool](/docs/tools/asloglatency) - asloglatency
Aerospike contains a number of settings that allow latency issues in a server to be diagnosed. This tool analyzes a logfile and displays the different components of a transaction.

#### [Aerospike Info](/docs/tools/asinfo) - asinfo
This low-level tool can make requests to an individual server over Aerospike's
command language. Useful for gathering statistics, and also setting a variety of tuning parameters. Often used by higher level scripts. Same commands can be used within an asadm shell to issue said commands against all nodes in a cluster.

#### [Aerospike Loader](/docs/tools/asloader) - asloader
This tool can help in migrating data from any other database to Aerospike. User can dump data from different databases in .DSV format and use this tool to parse and load them in Aerospike server.

Proceed to Aerospike Tools installation **[here](/docs/operations/install/tools)**.
