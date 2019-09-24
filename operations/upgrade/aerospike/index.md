---
title: Upgrading Aerospike
description: Learn how to upgrade Aerospike safely. 
styles:
  - /assets/styles/ui/steps.css
---

In order to get access to the latest features and bug fixes, you will need to upgrade Aerospike by following the given steps. 
To upgrade the cluster software, you will upgrade each server, one at a time which means no downtime for deploying the upgrades. 

{{#note}} 
For the purposes of performing a rolling upgrade, Aerospike supports mixed-version clusters. Aerospike does not, however, support long term use of clusters with nodes running different versions of the Aerospike database.
{{/note}}

{{#warn}}
Refer to the [**Special Upgrade instructions**](/docs/operations/upgrade/aerospike/special_upgrades/index.html) page for important information relevant to specific versions.
{{/warn}}

{{#steps}}

To upgrade the servers, begin by [downloading](/download) the latest build based on your OS. You can find the release notes explaining the features and fixes for the release on the same page too.

<br></br>

{{#steps-step 1 "Download the package to the node" markdown=true}}

Transfer the downloaded package to the node. Ensure that you download the OS specific package. 

{{#note}}
It is a good practice to quiesce a node prior to shutting it down or removing it from a cluster. Refer to the 
[Quiesce Node](/docs/operations/manage/cluster_mng/quiescing_node) page for further details (Enterprise Edition).
{{/note}}

{{/steps-step}} 

{{#steps-step 2 "(Optional) Stop the Aerospike service" markdown=true}}

For the installation to be complete, the Aerospike service will need to be restarted. You can either stop the service, upgrade the package and then start the service. Or you can do the upgrade and directly restart the service later. 

{{#warn}}
If you have namespaces which are configured to be only in-memory with no persistence ([`storage-engine`](/docs/reference/configuration/#storage-engine) memory), 
it is **required** to wait for migrations to be completed before moving on to the next node to avoid data loss. 
Refer to [Monitoring Migrations](/docs/operations/manage/migration#monitoring-migrations).
{{/warn}}


To stop the Aerospike service
```
$ sudo /etc/init.d/aerospike stop
```
```
# For Debian 10
$ sudo systemctl stop aerospike
```
{{/steps-step}}

{{#steps-step 3 "Upgrade to the latest package" markdown=true}}

The downloaded package for your Linux distribution will be named as below:

**aerospike-server-`<EDITION>`-`<VERSION>`**

where:
- `<VERSION>` - latest version of the build (typically something like 4.6.0.2)
- `<EDITION>` should be
    - **community** for the Community Edition
    - **enterprise** for the Enterprise Edition

{{#note}} If you are installing on Amazon EC2 with Amazon Linux, use the CentOS 6 installation (compatible with Amazon Linux).{{/note}}

The downloaded package will need to be un-compressed to get the server and the tools installation packages.

You may need to delete the old version if upgrading from Aerospike version prior to 3.3.x. To do so, look for the old version which should look like "aerospike-<EDITION>-server" and "aerospike-<EDITION>-tools". You can then remove the packages by using the command:

```
# For Centos and RedHat
$ sudo rpm -qa | grep aerospike
$ sudo rpm -e <PACKAGE_NAME>

```
```
# For Debian and Ubuntu
$ sudo dpkg -l | grep aerospike
$ sudo dpkg -r <PACKAGE_NAME>

```

Install the new packages.

### ** For Centos 6 (RHEL 6)**

```
$ tar xzf aerospike-server-<EDITION>-<VERSION>-el6.tgz 
$ cd aerospike-server-<EDITION>-<VERSION>-el6 

```
```
# If upgrading without removing the older package
$ sudo rpm -Uvh aerospike-server-<EDITION>-<VERSION>.el6.x86_64.rpm 
$ sudo rpm -Uvh aerospike-tools-<EDITION>-<VERSION>.el6.x86_64.rpm
```
```
# If upgrading after removing the older package
$ sudo rpm -ivh aerospike-server-<EDITION>-<VERSION>.el6.x86_64.rpm 
$ sudo rpm -ivh aerospike-tools-<EDITION>-<VERSION>.el6.x86_64.rpm
```

### ** For Debian 8+ and Ubuntu 14.04+**

```
# For Debian 10 
$ tar xzf aerospike-server-<EDITION>-<VERSION>-debian10.tgz
$ cd aerospike-server-<EDITION>-<VERSION>-debian10
$ sudo dpkg -i aerospike-server-<EDITION>-<VERSION>.debian10.x86_64.deb
$ sudo dpkg -i aerospike-tools-<EDITION>-<VERSION>.debian10.x86_64.deb

```
```
# For Debian 9
$ tar xzf aerospike-server-<EDITION>-<VERSION>-debian9.tgz
$ cd aerospike-server-<EDITION>-<VERSION>-debian9
$ sudo dpkg -i aerospike-server-<EDITION>-<VERSION>.debian9.x86_64.deb
$ sudo dpkg -i aerospike-tools-<EDITION>-<VERSION>.debian9.x86_64.deb

```
```
# For Debian 8
$ tar xzf aerospike-server-<EDITION>-<VERSION>-debian8.tgz
$ cd aerospike-server-<EDITION>-<VERSION>-debian8
$ sudo dpkg -i aerospike-server-<EDITION>-<VERSION>.debian8.x86_64.deb
$ sudo dpkg -i aerospike-tools-<EDITION>-<VERSION>.debian8.x86_64.deb

```
```
# For Ubuntu 18.04
$ tar xzf aerospike-server-<EDITION>-<VERSION>-ubuntu18.04.tgz
$ cd aerospike-server-<EDITION>-<VERSION>-ubuntu18.04
$ sudo dpkg -i aerospike-server-<EDITION>-<VERSION>.ubuntu18.04.x86_64.deb
$ sudo dpkg -i aerospike-tools-<EDITION>-<VERSION>.ubuntu18.04.x86_64.deb

```
```
# For Ubuntu 16.04
$ tar xzf aerospike-server-<EDITION>-<VERSION>-ubuntu16.04.tgz
$ cd aerospike-server-<EDITION>-<VERSION>-ubuntu16.04
$ sudo dpkg -i aerospike-server-<EDITION>-<VERSION>.ubuntu16.04.x86_64.deb
$ sudo dpkg -i aerospike-tools-<EDITION>-<VERSION>.ubuntu16.04.x86_64.deb

```
```
# For Ubuntu 14.04
$ tar xzf aerospike-server-<EDITION>-<VERSION>-ubuntu14.04.tgz
$ cd aerospike-server-<EDITION>-<VERSION>-ubuntu14.04
$ sudo dpkg -i aerospike-server-<EDITION>-<VERSION>.ubuntu14.04.x86_64.deb
$ sudo dpkg -i aerospike-tools-<EDITION>-<VERSION>.ubuntu14.04.x86_64.deb

```
{{/steps-step}}

{{#steps-step 4 "Start the Aerospike service" markdown=true}}

If you stopped Aerospike service, you can start that server and wait for the process to complete and for the server to confirm that the data is ready.

```
$ sudo /etc/init.d/aerospike start
```
```
# For systemd based OS
$ sudo systemctl start aerospike
```

If you did not stop the service, you still need to restart the Aerospike service in order for the upgrade to be complete. Stopping and starting the Aerospike service makes the node leave the cluster for a short time and then rejoin the cluster. This would cause data rebalancing (migrations) to trigger. 

```
$ sudo /etc/init.d/aerospike restart
```
```
# For systemd based OS
$ sudo systemctl restart aerospike
```

Prior to moving on to upgrading another node, make sure the node has re-joined the cluster. This can be done by checking the that:
1. cluster key is uniform across the cluster, see [`cluster_key`](/docs/reference/metrics#cluster_key)
2. cluster is of the expected size, see [`cluster_size`](/docs/reference/metrics#cluster_size)

This can be checked through [asadm](/docs/tools/asadm/index.html), [asinfo](/docs/tools/asinfo/index.html) or in the logs.

{{#warn}}
Some situations may require waiting for migrations to complete prior to moving on and stopping the next node. 

As of version 4.3, the [`cluster-stable`](/docs/reference/info/#cluster-stable) info command can be used to easily determine whether a cluster is stable.

For versions not supporting the [`cluster-stable`](/docs/reference/info/#cluster-stable) info command, the following conditions would guarantee that migrations 
have completed following a reclustering event (across all nodes in a cluster):<br>
<ul>
<li>[`migrate_allowed`](/docs/reference/metrics/#migrate_allowed) is `true`
<li>[`migrate_partitions_remaining`](/docs/reference/metrics#migrate_partitions_remaining) is `0`
</ul><br>
In order to programmatically check whether the cluster is at a stable state and migrations have finished, the following should be observed:
<ol>
<li>Wait until [`cluster_key`](/docs/reference/metrics#cluster_key) is uniform, the [`cluster_size`](/docs/reference/metrics/#cluster_size) is of expected size and [`migrate_allowed`](/docs/reference/metrics#migrate_allowed) is set to `true`
<li>Wait for [`migrate_partitions_remaining`](/docs/reference/metrics#migrate_partitions_remaining) to be `0`
<li>Check the [`cluster_key`](/docs/reference/metrics#cluster_key) is still the same as it was in step 1 and that the [`cluster_size`](/docs/reference/metrics/#cluster_size) did not change. Should these be different, loop over to step 1 and check again.
</ol><br><br>

Situations requiring a wait for migrations between nodes to complete include the following:<br>
<ul>
<li>namespaces which are configured to be in-memory only (no persistence, `storage-engine memory`)<br>
<li>specific version upgrades requiring persisted data to be deleted (for example version 4.2)<br>
<li>some use cases for clusters running the older cluster protocol (prior to version 3.13)<br>
</ul>
{{/warn}}

On completing the upgrade across all nodes in the cluster, you can confirm from the output of `asadm -e "info network"` if all the nodes have successfully upgraded to the correct version and that the cluster size is correct. You can look at `asadm -e "info node"` to check other statistics.

```
3 hosts in cluster: 192.168.120.xxx:3000,192.168.120.yyy:3000,192.168.120.zzz:3000
----------------------------------------------------------Network Information (2018-04-04 20:09:18 UTC)---------------------------------------------------------------------
Cluster          Node               Node                  Ip          Build        Cluster   Migrations       Cluster     Cluster        Principal       Client      Uptime   
   Name            .                 Id                   .             .            Size            .         Key       Integrity            .           Conns          .   
v24dc1    10.0.100.xxx:3000   *BB9DB3754005452   10.0.100.xxx:3000   E-4.0.0.4         3        0.000     1199DF2D0D95     True        BB9DB3754005452        2     23:12:29   
v24dc1    10.0.100.yyy:3000   BB9AC3883005452    10.0.100.yyy:3000   E-4.0.0.4         3        0.000     1199DF2D0D95     True        BB9DB3754005452        2     27:08:05   
v24dc1    10.0.100.zzz:3000   BB9117BC5005452    10.0.100.zzz:3000   E-4.0.0.4         3        0.000     1199DF2D0D95     True        BB9DB3754005452        2     19:56:39   
Number of rows: 3
```


{{#note}}`service ready: soon there will be cake!` will be logged once a node is available. The logs can be tailed for "cake" to check for this line:

```
$ sudo tail -f /var/log/aerospike/aerospike.log | grep "cake"
```
{{/note}}

{{/steps-step}}


{{/steps}}
