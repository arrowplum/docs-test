---
title: Cluster Protocols Upgrade in 3.13
description: Operational procedure for cluster protocols upgrade in 3.13.
---

{{#warn}}
3.13 is a REQUIRED jump-version upgrade for any future Aerospike releases  - deployments on any prior version MUST first upgrade to 3.13, make the cluster protocols switch-over as described in this document, before proceeding to any subsequent version upgrade. An upgrade to any future version which skips over 3.13 or did not perform the cluster protocols switch-over while on 3.13 WILL NOT START UP. Enterprise customers can reach out to Aerospike Support for any specific questions about this process.
{{/warn}}

The full cluster protocols switch-over in version 3.13 is a 3 step process:
<ol>
**<li>
Upgrade nodes in the cluster to 3.13 Aerospike daemon, as usual.**
**<li>
After migrations have completed, schedule a time to run the provided [script](/docs/operations/upgrade/cluster_to_3_13#a-script-recommended-) to dynamically change the heartbeat protocol to v3 and paxos-protocol to v5.**
**<li>
Upgrade to version 3.14 or above.
When performing this upgrade, the paxos-protocol and heartbeat protocol do not need to be explicitly set in the aerospike configuration file as paxos-protocol defaults to v5 and heartbeat protocol defaults to v3 in versions 3.14 and above. Therefore those 2 configuration parameters can be removed from the configuration file.**
<br>
{{#info}}
If planning to remain on version 3.13.x, the static aerospike configuration file (aerospike.conf) should be updated with the required paxos-protocol set to v5 in the service stanza and the heartbeat protocol set to v3 in the network stanza.
It is a recommended good practice to perform a rolling restart on each node in the cluster to ensure the configuration files have been changed correctly. This will help to avoid potential surprises if a node was to be restarted at a later time. Restart one node at a time, waiting for the cluster to reform before moving to the next one.
{{/info}}

- For rack-aware setup, check the [rack-aware](/docs/operations/upgrade/cluster_to_3_13/rackaware) instructions.
- It is also recommended to go through the [FAQ](/docs/operations/upgrade/cluster_to_3_13/faq) prior to proceeding with the cluster protocols switch.

## 1- Upgrade the Aerospike daemon to 3.13

The procedure to upgrade to version 3.13 daemon is similar to typical version upgrades. Nodes in the cluster should be upgraded one at a time, which will not cause any downtime for the cluster. For the purposes of performing a rolling upgrade, Aerospike supports mixed-version clusters. Aerospike does not, however, support long term use of clusters with nodes running different versions of the Aerospike database. Please also see [Upgrade Aerospike Daemon](/docs/operations/upgrade/aerospike).

<b>Enterprise Users: To upgrade the servers, begin by [downloading](/enterprise/download/server/notes.html#3.13.0.11) the latest 3.13 build for your OS.<br>
Community Users : To upgrade the servers, begin by [downloading](/artifacts/aerospike-server-community/3.13.0.11/) the latest 3.13 build for your OS.
</b>

{{#note}}
 Please perform the following steps on each node in the cluster. 
{{/note}}

{{#note}}
Depending on the nature of the write workload against the cluster, it may be necessary to wait for migrations to complete before proceeding to upgrade the next node. Enterprise licensees should contact aerospike support for any questions regarding this process. Community users can post questions on the [forum](https://discuss.aerospike.com). Once on the new cluster protocols, this will not be necessary for subsequent rolling upgrade/restarts.
{{/note}}


#### **1.1- Download and install the package on the node**

Transfer the downloaded package to the node. Ensure that you download the OS specific package. 

{{#note}} If you are installing on Amazon EC2 with Amazon Linux, use the CentOS 6 installation (compatible with Amazon Linux).{{/note}}

Untar the package and run the asinstall script. Refer to [Install Aerospike](/docs/operations/install) for detailed installation instuctions of the downloaded package. 

Verify that the right version of the binary is installed:

```asciidoc
$ asd --version
Aerospike Enterprise Edition build 3.13.0.3
```

#### **1.2- Restart the Aerospike service**

Restart the Aerospike service in order for the upgrade to be complete.

```asciidoc
$ sudo /etc/init.d/aerospike restart
```

To confirm that the server is available, tail the server logs and look for “service ready: soon there will be cake!”.

```asciidoc
$ sudo tail -f /var/log/aerospike/aerospike.log | grep "cake"
```

{{#warn}}
Restarting the Aerospike service makes the node leave the cluster for a short time and then rejoin the cluster. This would cause data rebalancing (migrations) to trigger. Depending on your write workload type, wait for migrations to complete, then move on to the next node to continue with the upgrade. If you have namespaces which are configured to be only in-memory with no persistence, it is **required** to wait for migrations to be completed before moving on to the next node to avoid data loss.
{{/warn}}


#### **1.3- Validate the upgrade to 3.13**

After upgrading each node, you can confirm from the output of `asadm -e info` whether all the nodes have successfully been upgraded to 3.13 version and cluster size is shown correctly.

```asciidoc
$ asadm -e info
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                         Node               Node                   Ip        Build   Cluster           Cluster     Cluster         Principal   Client     Uptime   
                            .                 Id                    .            .      Size               Key   Integrity                 .    Conns          .   
192.168.1.177:3000              *BB99E8005270008   192.168.1.177:3000   E-3.13.0.3         3   C8A6F6E3F51A8D3   True        BB99E8005270008        7   00:19:26   
192.168.1.191:3000              BB90A09E3270008    192.168.1.191:3000   E-3.13.0.3         3   C8A6F6E3F51A8D3   True        BB99E8005270008        7   03:01:21   
192.168.1.192:3000              BB9235677270008    192.168.1.192:3000   E-3.13.0.3         3   C8A6F6E3F51A8D3   True        BB99E8005270008        7   00:05:22   
Number of rows: 3

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace              Node   Avail%   Evictions                 Master                Replica     Repl     Stop             Pending       Disk    Disk     HWM         Mem     Mem    HWM      Stop
        .                 .        .           .   (Objects,Tombstones)   (Objects,Tombstones)   Factor   Writes            Migrates       Used   Used%   Disk%        Used   Used%   Mem%   Writes%
        .                 .        .           .                      .                      .        .        .             (tx,rx)          .       .       .           .       .      .         .
test     192.168.1.177:3000   N/E        0.000     (118.424 K, 0.000)     (114.573 K, 0.000)     2        false    (0.000,  0.000)          N/E   N/E     50      29.553 MB   1       60     90 
test     192.168.1.191:3000   N/E        0.000     (114.931 K, 0.000)     (120.095 K, 0.000)     2        false    (0.000,  0.000)          N/E   N/E     50      29.810 MB   1       60     90
test     192.168.1.192:3000   N/E        0.000     (117.678 K, 0.000)     (116.365 K, 0.000)     2        false    (0.000,  0.000)          N/E   N/E     50      29.686 MB   1       60     90 
test                                     0.000     (351.033 K, 0.000)     (351.033 K, 0.000)                       (0.000,  0.000)     0.000 B                    89.049 MB
Number of rows: 4
```


## 2- Upgrade the heartbeat protocol and paxos-protocol versions

This part consists of the following steps:

- Increase heartbeat timeout.
- Upgrade heartbeat protocol version.
- Upgrade paxos-protocol version.
- Set heartbeat timeout back to its initial value.

{{#warn}}
Once the paxos-protocol setting has been set to v5, the value cannot be changed without a complete cluster shutdown.
The setting would need to be statically set to v2 or v3 in aerospike.conf for a downgrade. This change cannot be done dynamically on a cluster running on paxos-protocol v5.
{{/warn}}

**Two alternate methods are available for this part:**

A- Using a provided script.<br>
B- Applying the dynamic configuration changes through manual steps.<br>

### A- Script (**Recommended**)

This [script](https://www.aerospike.com/artifacts/operations/switch_to_v5) will actually perform all those steps. Simply download the linked file, 
make sure the executable bit is set and run it as such from one of the node in the cluster, or by specifying a host ip and port:

{{#note}}
Use wget as displayed in the example, if the browser does not download the switch_to_v5 link properly. 
{{/note}}

```asciidoc
$ wget -O switch_to_v5 'https://www.aerospike.com/artifacts/operations/switch_to_v5' 
--2017-11-16 16:05:23--  https://www.aerospike.com/artifacts/operations/switch_to_v5
Resolving www.aerospike.com... 172.120.0.1, 172.120.0.2
Connecting to www.aerospike.com|172.120.0.1|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 199372 (195K)
Saving to: `switch_to_v5'

100%[==================================================================================================================================================================>] 199,372     --.-K/s   in 0.08s   

2017-11-16 16:05:23 (2.43 MB/s) - `switch_to_v5' saved [199372/199372]

$ ./switch_to_v5 -h 172.16.191.174 -p 3000
Found 3 nodes
Online:  172.16.191.174:3000, 172.16.191.175:3000, 172.16.191.176:3000
please confirm cluster nodes (yes/no): yes
cluster confirmed ...
cluster is stable ...
changed heartbeat timeout to 10000 on all the nodes
changed heartbeat protocol to v3 on all the nodes
allowing cluster some time to settle down
cluster is stable ...
WARNING: about to set paxos-protocol to v5, this is irreversible. 
do you want to go ahead (yes/no): yes
setting paxos-protocol to v5 ...
setting heartbeat timeout to previous value
reverting heartbeat.timeout on nodes: ['172.16.191.174:3000', '172.16.191.175:3000', '172.16.191.176:3000']

successfully completed setting paxos-protocol to v5
```

For the full list of options available, including running with authentication or tls, check the help:

```asciidoc
./switch_to_v5 --help
```

{{#warn}}
For secure auth or tls enabled clusters, make sure a version of the Aerospike tools package greater than 3.11 is installed to ensure the relevant dependencies necessary for this script are available. 
{{/warn}}

### B- Manual steps for dynamically updating the configuration

In order to proceed manually with the change, follow the steps below.

{{#note}}
Please note that each of the below commands have to be run from the "asadm" prompt which ensures that the command runs across the cluster.
{{/note}}


#### **2.1- Ensure the cluster is stable**

- The Migrate column indicates zeros
- The cluster is in its correct size
- Each node on the cluster is on build 3.13

"asadm -e info" shows the below information to validate the above mentioned stability parameters of the cluster:

```asciidoc
$ asadm -e info
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                         Node               Node                   Ip        Build   Cluster           Cluster     Cluster         Principal   Client     Uptime   
                            .                 Id                    .            .      Size               Key   Integrity                 .    Conns          .   
192.168.1.177:3000              *BB99E8005270008   192.168.1.177:3000   E-3.13.0.3         3   C8A6F6E3F51A8D3   True        BB99E8005270008        7   00:19:31   
192.168.1.191:3000              BB90A09E3270008    192.168.1.191:3000   E-3.13.0.3         3   C8A6F6E3F51A8D3   True        BB99E8005270008        7   03:01:26   
192.168.1.192:3000              BB9235677270008    192.168.1.192:3000   E-3.13.0.3         3   C8A6F6E3F51A8D3   True        BB99E8005270008        7   00:05:27   
Number of rows: 3

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace              Node   Avail%   Evictions                 Master                Replica     Repl     Stop             Pending       Disk    Disk     HWM         Mem     Mem    HWM      Stop   
        .                 .        .           .   (Objects,Tombstones)   (Objects,Tombstones)   Factor   Writes            Migrates       Used   Used%   Disk%        Used   Used%   Mem%   Writes%   
        .                 .        .           .                      .                      .        .        .             (tx,rx)          .       .       .           .       .      .         .   
test     192.168.1.177:3000   N/E        0.000     (118.424 K, 0.000)     (114.573 K, 0.000)     2        false    (0.000,  0.000)          N/E   N/E     50      29.553 MB   1       60     90        
test     192.168.1.191:3000   N/E        0.000     (114.931 K, 0.000)     (120.095 K, 0.000)     2        false    (0.000,  0.000)          N/E   N/E     50      29.810 MB   1       60     90        
test     192.168.1.192:3000   N/E        0.000     (117.678 K, 0.000)     (116.365 K, 0.000)     2        false    (0.000,  0.000)          N/E   N/E     50      29.686 MB   1       60     90        
test                                     0.000     (351.033 K, 0.000)     (351.033 K, 0.000)                       (0.000,  0.000)     0.000 B                    89.049 MB                            
Number of rows: 4
```


#### **2.2- Extend heartbeat timeout**

For every node, extend heartbeat.timeout so we have 30 minutes to execute the cluster protocols switch-over:

(heartbeat.timeout x heartbeat.interval = 1800000)

Assuming the heartbeat.interval is set to the default 150ms, to gain 30min of switch time we need to set the heartbeat.timeout to 12000.

```asciidoc
Admin> asinfo -v "set-config:context=network;heartbeat.timeout=12000"
192.168.1.177:3000 (192.168.1.177) returned:
ok

192.168.1.191:3000 (192.168.1.191) returned:
ok

192.168.1.192:3000 (192.168.1.192) returned:
ok
```

To verify the heartbeat timeout is set to 12000, look for the following message in aerospike.log on each node:

```asciidoc
May 14 2017 23:35:43 GMT: INFO (info): (thr_info.c:3491) config-set command completed: params context=network;heartbeat.timeout=12000
```

You can also look at the configuration parameter:

```asciidoc
Admin> show config like heartbeat.timeout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Configuration~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE             :   192.168.1.177:3000   192.168.1.191:3000   192.168.1.192:3000   
heartbeat.timeout:   12000                12000                12000                           
```

#### **2.3- Change the heartbeat protocol version to v3**

For every node, change the heartbeat protocol - set-config:context=network;heartbeat.protocol=v3:

```asciidoc
Admin> asinfo -v "set-config:context=network;heartbeat.protocol=v3"
192.168.1.177:3000 (192.168.1.177) returned:
ok

192.168.1.191:3000 (192.168.1.191) returned:
ok

192.168.1.192:3000 (192.168.1.192) returned:
ok
```

Verify that the heartbeat protocol version changed to v3:

```asciidoc
Admin> show config like heartbeat.protocol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Configuration~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE              :   192.168.1.177:3000   192.168.1.191:3000   192.168.1.192:3000   
heartbeat.protocol:   v3                   v3                   v3                              
```

#### **2.4- Change the paxos-protocol version to v5**

```asciidoc
Admin> asinfo -v "set-config:context=service;paxos-protocol=v5"
192.168.1.177:3000 (192.168.1.177) returned:
ok

192.168.1.191:3000 (192.168.1.191) returned:
ok

192:168.1.192:3000 (192.168.1.192) returned:
ok
```

The following log messages are logged if the above command is successful on each node:

```asciidoc
May 15 2017 14:31:47 GMT: INFO (info): (thr_info.c:2367) Changing value of paxos-protocol version to v5
May 15 2017 14:31:47 GMT: INFO (info): (thr_info.c:3491) config-set command completed: params context=service;paxos-protocol=v5
```

You could also verify by looking at the configuration parameter as below:

```asciidoc
Admin> show config like paxos-protocol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Service Configuration~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE          :   192.168.1.177:3000   192.168.1.191:3000   192.168.1.192:3000   
paxos-protocol:   v5                   v5                   v5                              
```

{{#warn}}
Please note that this change from v3 version of paxos-protocol to v5 is NOT dynamically reversible i.e cannot be changed back to v3 during runtime.
{{/warn}}


#### **2.5- Restore heartbeat.timeout to original value**

Assuming that the original value of heartbeat.timeout was 10 (default), the following command restores the heartbeat.timeout to its original value:

```asciidoc
Admin> asinfo -v "set-config:context=network;heartbeat.timeout=10"
192.168.1.177:3000 (192.168.1.177) returned:
ok

192.168.1.191:3000 (192.168.1.191) returned:
ok

192.168.1.192:3000 (192.168.1.192) returned:
ok
```

A successful execution of the above command logs the following lines in aerospike.log:

```asciidoc
May 15 2017 15:15:34 GMT: INFO (hb): (hb.c:3665) Changing value of heartbeat.timeout from 12000 to 10 
May 15 2017 15:15:34 GMT: INFO (info): (thr_info.c:3491) config-set command completed: params context=network;heartbeat.timeout=10
```

You can also verify by looking into the configuration parameters:

```asciidoc
Admin> show config like heartbeat.timeout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Configuration~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE             :   192.168.1.177:3000   192.168.1.191:3000   192.168.1.192:3000   
heartbeat.timeout:   10                   10                   10                              
 ```

#### **2.6- Ensure the cluster is stable**

Make sure the cluster is stable. There should be no migrations happening.


## 3- Update each node's static configuration file.

Once the cluster protocols change has been effected, update the static configuration file (aerospike.conf).

If planning to remain on version 3.13.x and wanting to permanently effect the protocol changes, the aerospike configuration file(aerospike.conf) has to be updated on each node to ensure nodes would join the cluster after restart.
It is a recommended good practice to perform a rolling restart on each node in the cluster to ensure the configuration files have been changed correctly. This will help to avoid potential surprises if a node was to be restarted at a later time. Restart one node at a time, waiting for the cluster to reform before moving to the next one. 

If not planning to remain on version 3.13.x and are planning to upgrade directly from version 3.13.x to version 3.14 or above there is no need to explicitly set the cluster protocols in the aerospike configuration file as paxos-protocol defaults to v5 and heartbeat protocol defaults to v3. Those 2 configuration parameters can be removed from the configuration file.

The only difference between 3.13.0.3 and version 3.14.0.2 is that 3.14.0.2 defaults to the new paxos-protocol and heartbeat protocol settings, removes depricated code, and removes unused code. 

### A- Upgrade from version 3.13.x to 3.14 or above (**Recommended**)

After upgrading to version 3.13.x to directly upgrade to version 3.14/3.15 or above, update the configuration file to remove the paxos-protocol and heartbeat.protocol configuration parameters and simply proceed with a rolling upgrade of each node.

{{#note}}
As the cluster has been upgraded to the new cluster protocols, it is not necessary to wait for migrations to complete anymore, regardless of the nature of the write workload.
{{/note}}

### B- Manual configuration file update, if remaining on version 3.13.x

#### **3.1- Update heartbeat protocol to v3 and paxos-protocol to v5 in the configuration file**

In the configuration file(default location:/etc/aerospike/aerospike.conf), update the paxos-protocol version to v5 in the service section and the heartbeat protocol version to v3 in the network section:

```asciidoc
service {
	...
	paxos-protocol v5
	...
}

network {
    ...
   	heartbeat {	
		...
		protocol v3
		...
	}
    ...
}
...
```

#### **3.2- Rolling restart each node in the cluster**

This step is not required but is a recommended good practice to ensure the configuration files have been changed correctly and avoid potential surprises if a node was to be restarted at a later time. Restart one node at a time, waiting for the cluster to reform before moving to the next one. 

{{#note}}
As the cluster has been upgraded to the new cluster protocols, it is not necessary to wait for migrations to complete anymore, regardless of the nature of the write workload.
{{/note}}
