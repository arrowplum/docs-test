---
title: Upgrading Aerospike 2 to 3
description: Learn the proper procedures to ensure there is no downtime required to upgrade from previous Aerospike versions.
styles:
  - /assets/styles/ui/steps.css
---

The storage format and client wire protocol of Aerospike 2 is fully compatible
with Aerospike 3. Aerospike 3 supports Aerospike 2's heartbeat and paxos
protocols, but by default those protocols may be of a newer version and must be configured to
be the same as your existing cluster. Though the Aerospike 2 configuration
files are still compatible with Aerospike 3, we encourage you to update the old paths and names
to Aerospike 3 paths and names. These steps ensure that there is no downtime required to
upgrade from Aerospike 2 to Aerospike 3 if you are upgrading to Aerospike 3.10+. For upgrading to a version prior to 3.10, if you are using mesh configuration, this procedure would require downtime. When you start Aerospike 3 for the first time, it performs a cold start. The following procedures provide the required steps for the upgrade process.

{{#note}}
Please contact [Aerospike Support](http://support.aerospike.com) if you need
clarification on any of the following steps.
{{/note}}

{{#warn}}
If you are upgrading from a version prior to **2.7.4** and using **XDR**, please
contact [Aerospike Support](http://support.aerospike.com) for special instructions for dealing with a change
in how **XDR** handles zero and negative one TTLs.
{{/warn}}

{{#steps}}

{{#steps-step 1 "(Optional) Aerospike 2 Backup" markdown=true}}
Before you upgrade the node, make a backup of the data.

1. Before performing the backup, verify that the cluster isn't migrating data. If it is migrating, allow the cluster to finish before
   proceeding.

   Use the following command to verify migrations are complete:
   ```bash
   clmonitor -e "stat -v migrate_progress"
   ```
   Migrations are complete when all `migrate_progress_recv` and
   `migrate_progress_send` values are 0.
2. If there are no active migrations, backup the cluster.

    For each namespace run:
    ```bash
    clbackup2 -d <path/to/backup/namespace> -c -b -n <namespace-name>
    ```
{{/steps-step}}

{{#steps-step 2 "Backup Configuration File for Aerospike 2" markdown=true}}
Create a backup of the current configuration file:
```
sudo cp /etc/citrusleaf/citrusleaf.conf /etc/citrusleaf/citrusleaf.conf.bac
```
{{/steps-step}}

{{#steps-step 3 "Configuration Compatibility Changes" markdown=true}}
{{#note}}
This step is only needed if you plan to perform a rolling upgrade to Aerospike 3. If you are going to take the entire cluster down to upgrade, then
you may bypass this step.
{{/note}}

1. Create another copy of citrusleaf.conf.  You will modify the copy to be Aerospike 3 compatible: 
   ```
   sudo cp /etc/citrusleaf/citrusleaf.conf /etc/citrusleaf/aerospike.conf
   ```
2. By default, Aerospike 3 uses `paxos-protocol v3`, `heartbeat-protocol v2` (for version prior to 3.10+) and `heartbeat-protocol v3` (for 3.10+),
   which are likely different from your Aerospike 2 configuration.<br/><br/>
   First, determine the version of the heartbeat and paxos protocols in your existing cluster:<br/>
   ```
   clmonitor -e 'printconfig -v *protocol'
   ```
   If you do not have a 'paxos-protocol' or a 'heartbeat-protocol', add
   them to the configuration file as shown in the examples:

3. Open `/etc/citrusleaf/aerospike.conf` for editing.

4. In the `service` context, change the value of `pidfile` to
   `/var/run/aerospike/asd.pid`. Update `paxos-protocol` to the
   paxos protocol version currently in use by your existing cluster.
   ```
   e.g. paxos-protocol v1

   ```

5. In the `logging` context, change the value of `file` to:
   `/var/log/aerospike/aerospike.log`

6. In the `network` context, `heartbeat` sub-context, update `protocol` to the
   heartbeat protocol version in use by your existing cluster:
   ```
   e.g. protocol v1

   ```

7. In the `namespace` contexts:
   - If you use namespaces persisted to file and the files are a sub-directory
     of `/opt/citrusleaf/` change them to be a child of `/opt/aerospike` and move the files to the new location.

8. If the `storage-engine` was configured to `ssd` then change it to `device`.

9. In the `xdr` context, make the following changes:
   - Change `namedpipe-path` to `/tmp/xdr_pipe`
   - Change `digestlog-path` to `/etc/aerospike/digestlog`
   - Change `errorlog-path` to `/var/log/asxdr.log`

 {{#note}}
 If upgrading to server version 3.8+, follow the configuration recommentations provided on "[Upgrade XDR to 3.8](/docs/operations/upgrade/xdr_to_3_8)".
 {{/note}}

10. Change old sizes/times to human readable form, e.g., `24G` instead of
    `25769803776`

 **Before**:

   ```ruby
   service {
       ...
       pidfile <old location>
   }
   ...
   logging {
       file /var/log/citrusleaf.log {
           context any info
       }
       ...
   }
   ...
   namespace <namespace-name> {
       ...
       storage-engine ssd {
           ...
       }
       ...
   }
   ...
   ```

   **After**:

   ```ruby
   service {
       ...
       pidfile /var/run/aerospike/asd.pid
       paxos-protocol <same version as existing cluster>
   }
   ...
   logging {
       file /var/log/aerospike/aerospike.log {
           context any info
       }
       ...
   }
   ...
   network {
       ...
       heartbeat {
           ...
           protocol <same version as existing cluster>
       }
       ...
   }
   ...
   xdr {
       ...
       namedpipe-path  /tmp/xdr_pipe
       digestlog-path  /etc/aerospike/digestlog
       errorlog-path   /var/log/asxdr.log
       ...
   }
   ...
   namespace <namespace-name> {
       ...
       storage-engine device {
           ...
       }
       ...
   }
   ...
   ```


{{/steps-step}}

{{#steps-step 4 "Install Aerospike 3" markdown=true}}
{{#note}}
If you plan to bring the entire cluster down to upgrade then you may perform the
following procedures on all nodes at the same time instead of one node at a
time.
{{/note}}

On each node, one node at a time:
1. Stop Aerospike 2:
   ```bash
   sudo /etc/init.d/citrusleaf stop
   ```
1. Uninstall Aerospike 2 server and tools:
   - On Redhat/Centos distributions:<br>
	 ```bash
	 sudo rpm -e citrusleaf-server
	 sudo rpm -e citrusleaf-tools
	 ```
   - On Debian/Ubuntu distributions:<br>
	 ```bash
	 sudo dpkg -r citrusleaf-server
	 sudo dpkg -r citrusleaf-tools
	 ```
1. Perform the **Download Aerospike** and **Install Aerospike** procedures detailed
   for your operating system in the
   [Installation Documentation](/docs/operations/install).

2. Copy the Aerospike 3 compatible configuration file to
   `/etc/aerospike/aerospike.conf`:
   ```
   cp /etc/citrusleaf/aerospike.conf /etc/aerospike/aerospike.conf
   ```

3. Perform the **Run Aerospike** procedures detailed for your operating system in
   the [Installation Documentation](/docs/operations/install). When you start Aerospike after upgrading from version 2 to version 3, it performs a cold-start.
   
   {{#info}}
   Note: You may see warnings in the Aerospike's log explaining that some
   parameters copied from your old configuration are now deprecated in
   Aerospike 3. Make sure to confirm their corresponding usage in Aerospike 3 by
   confirming in the [Configuration Reference](/docs/reference/configuration).
   {{/info}}

4. Wait for Aerospike's service port to open (port 3000) before continuing.
   When the service port is open, the following command will return "OK":
   ```bash
   asinfo -v STATUS
   ```

5. If your server contains any namespaces that are **in-memory without
   persistence** then you must wait for migrations to complete before continuing
   to the next node to prevent data loss. Run the following command to verify
   migrations have completed:
   ```bash
   # Migrations are complete when all `migrate_progress_recv` and
   # `migrate_progress_send` values return 0.
   asadm -e "show statistics like migrate_progress"
   ```

   If all namespaces are **persisted**, you may continue to upgrade without any data loss.
   {{#warn}}
   If writes are still active on the cluster, and you choose to not wait for
   migrations to complete between upgrades, some records may become
   inconsistent during the upgrade process. When upgrades are complete and
   migrations finish, the data will become consistent again and reflect the
   latest version of the records.
   {{/warn}}
{{/steps-step}}

{{#steps-step 5 "Set Paxos and Heartbeat Protocols" markdown=true}}
{{#note}}
This step is only required if you perform step 3.  Perform these steps only after you have upgraded all nodes in the cluster. Note that these steps are valid only for multicast setup. For mesh setup, you should skip Step 3 and startup Aerospike 3.x directly with the default protocol versions.
{{/note}}

Before, we configured paxos and heartbeat protocols to be that of Aerospike 2.
Without `paxos-protocol v3`, we cannot use many Aerospike 3 features such as
Secondary Indexes. The following procedures dynamically change the paxos
and heartbeat protocols.
{{#warn}}
Be careful when running these steps, performing them in the wrong order may
cause problems with the clustering algorithms.
{{/warn}}

1. Disable heartbeat on all nodes:
   ```
   asadm -e 'asinfo -v "set-config:context=network;heartbeat.protocol=none"'
   ```

 {{#note}}
 Skip this step if upgrading to version 3.10+.
 {{/note}}

2. Disable Paxos messaging on all nodes:

   ```
   asadm -e 'asinfo -v "set-config:context=service;paxos-protocol=none"'
   ```
 {{#note}}
 Skip this step if upgrading to version 3.10+.
 {{/note}}

3. If your cluster has 31 or more nodes, set the new maximum cluster size on all nodes in the cluster:
   <br/>
   The maximum value allowed is 128. The actual maximum cluster size is one less
   than the number given.<br/>
   That is, the default of 32 actually is limited to 31 nodes.

 {{#note}}
 For Aerospike Community Edition Server, the maximum number of nodes in a cluster is limited to 31 for versions prior to 4.0 and limited to 8 for versions 4.0 and above. 
 {{/note}}

   Changing this value in the future requires the heartbeat to be stopped again.
   Heartbeat packets are fixed length regardless of cluster size, limiting this
   value reduced the network load generated by heartbeats.

   ```
   asadm -e 'asinfo -v "config-set:context=service;paxos-max-cluster-size=<Expected Size + 10 (room for growth)>"'
   ```
 {{#note}}
 If upgrading to server version 3.10+, this step should be skipped as with heartbeat protocol v3, the cluster sizes can be changed dynamically and `paxos-max-cluster-size` configuration is deprecated.
 {{/note}}

4. Enable Paxos protocol v3 on all nodes:

   ```
   asadm -e 'asinfo -v "config-set:context=service;paxos-protocol=v3"'
   ```

5. Enable Heartbeat protocol v2 (or v3 if upgraded to 3.10+) on all nodes:

   ```
   asadm -e 'asinfo -v "config-set:context=network;heartbeat.protocol=v2"'
   ```

6. You no longer need to include the **paxos-protocol** or **protocol** in **/etc/aerospike/aerospike.conf** because Aerospike 3 uses the default values on all nodes.  Remove these values from **/etc/aerospike/aerospike.conf**. Remove the configurations for
   **paxos-protocol** in the service context and **protocol** in the network.heartbeat context. Add your configuration for
   **paxos-max-cluster-size** into the service context if your cluster size is 31 or more nodes for server version prior to 3.10.

   ```ruby
   service {
       ...
	   # remove: paxos-protocol <same version as existing cluster>
       # If your cluster has 31 or more nodes, add: paxos-max-cluster-size <Expected Size + 10 (room for growth)>
   }
   ...
   logging {
       file /var/log/aerospike/aerospike.log {
           context any info
       }
       ...
   }
   ...
   network {
       ...
       heartbeat {
           ...
           # remove: protocol <same version as existing cluster>
       }
       ...
   }
   ...
   ```
{{/steps-step}}
{{/steps}}

