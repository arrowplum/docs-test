---
title: Upgrading Hardware
description: Learn how to upgrade Aerospike cluster hardware safely without interrupting service. 
---

To upgrade the cluster hardware, you can take down a single node at a time.

In the event that you are upgrading your hardware, note that by design, Aerospike distributes the data evenly across all nodes in the cluster, so we assume that all of the servers have equivalent capacity.  For best performance, we recommend that all nodes in a cluster have equal storage.  If you add a new node with a larger SSD, the full capacity of that SSD will not be used effectively unless the other nodes are upgraded to have the same capacity.

**1. Before beginning to upgrade a server**, you must determine the correct type of restart to ensure that data migrates correctly after the upgrade.

**2. Stop the server that is to be upgraded:**
	
{{#info}}If you are upgrading your whole cluster, before you stop a server, you must be sure that the cluster is not doing data migration (as described below). Interrupting a cluster during data migration may cause data loss and may result in your database NOT being fully ACID-compliant.{{/info}}

```
$ sudo /etc/init.d/aerospike stop
```

**3. Upgrade the hardware.** We recommend that you do a power reset whenever you change your hadware, even if your hardware supports hot swapping of SSDs.

**4. If you add any SSDs, you need to prepare the new drives.**  New SSD drives must be prepared/installed before use. This process erases the drives using DD as described in the [SSD Initialization](/docs/operations/plan/ssd/ssd_init.html) section.  
	
{{#info}}Version 3.3.5 onwards, you do not need to dd the existing devices in a namespace if you are adding a new disk to the machine. You can add the new disk with fast restart. Please dd the new disk before adding to the namespace to be on safer side.{{/info}}

{{#info}}{{#markdown}}Before version 3.3.5 - If you add one or more SSDs, you must do DD to ALL of the drives (including the existing drives) to zero them out – this process takes hours to do, but can do in parallel on multiple drives.

If you don't DD the existing (previously installed) drives, then there can be odd errors and inconsistent behavior.{{/markdown}}{{/info}}

**5. Adjust your configuration file to use any expanded memory or SSD storage.**  To do this, review the installation instructions to make sure that your new hardware is correctly configured as described in the [instructions](/docs/operations/configure) for configuring storage for your system.

**6. Start that server** and wait for the process to complete and for the server to confirm that the data is ready:

```
$ sudo /etc/init.d/aerospike start
```

To confirm that the server is available, tail the server logs and look for “service ready: soon there will be cake!” :

```
$ sudo tail -f /var/log/aerospike/aerospike.log | grep "cake"
```

**7. Before upgrading another server, make sure that the data migration / data rebalancing process in the cluster is complete.** To check the migration status use the command:

```
$ asadm
Monitor> info
```

The output from the info command looks something like this.

```
Admin> info 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
               Node               Node                    Ip                  Build   Cluster            Cluster     Cluster         Principal   Client     Uptime   
                  .                 Id                     .                      .      Size                Key   Integrity                 .    Conns          .   
172.16.146.135:3000   *BB907DF26565000   172.16.146.135:3000                E-3.8.3         1   9203DDDCEBEE7D97   True        BB907DF26565000        1   01:52:04   
Number of rows: 1

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace                  Node   Avail%   Evictions     Master   Replica     Repl     Stop     Pending        Disk    Disk     HWM        Mem     Mem    HWM      Stop   
        .                     .        .           .    Objects   Objects   Factor   Writes    Migrates        Used   Used%   Disk%       Used   Used%   Mem%   Writes%   
        .                     .        .           .          .         .        .        .   (tx%,rx%)           .       .       .          .       .      .         .   
test        172.16.146.135:3000   99         0.000     82.935 K   0.000     1        false    (0,0)       10.124 MB   1       50      5.062 MB   1       60     90        
test                                         0.000     82.935 K   0.000                       (0,0)       10.124 MB                   5.062 MB                            
Number of rows: 2
```

When data migration is complete, the Migrates column will show (0,0) for all nodes.

{{#todo}}Add better output with more nodes.{{/todo}}
	

{{#info}}{{#markdown}}Do not upgrade another node in your cluster until the Migrates column shows zeroes. Interrupting data migration may cause data loss.

If you are upgrading a large cluster, you may begin upgrading the next server after a delay of 30-60 seconds, without allowing the data migration to complete.  This method may not be fully ACID compliant.{{/markdown}}{{/info}}

### Reconfiguring for Additional RAM

When adding more RAM, remember to adjust the high and low water mark settings appropriately, to ensure sizing are within normal parameters.

Not doing proper adjustments may result in available memory percentage going sideways unexpectedly.

{{#todo}}Add link for adjusting watermark.{{/todo}}

This error (from “/var/log/aerospike/aerospike.log“) means that the node is out of room to write:

```
Sep 13 2012 22:01:24 GMT: INFO (rw): (base/thr_rw.c:2300) writing pickled failed 8 for digest d480de145a6fac04
Sep 13 2012 22:01:24 GMT: INFO (rw): (base/thr_rw.c:2300) writing pickled failed 8 for digest 55fa14153280be49
Sep 13 2012 22:01:24 GMT: INFO (rw): (base/thr_rw.c:2300) writing pickled failed 8 for digest c9c30cae39037fe5
Sep 13 2012 22:01:24 GMT: INFO (rw): (base/thr_rw.c:2300) writing pickled failed 8 for digest 7f44a33bbc21b2d4
Sep 13 2012 22:01:24 GMT: INFO (rw): (base/thr_rw.c:2300) writing pickled failed 8 for digest b397d7616d0ca1cd
```
