---
title: Upgrading XDR to version 3.8
description: Instruction for upgrading XDR to version 3.8 where the asxdr process is merged with the asd process.
styles:
  - /assets/styles/ui/steps.css
---

In version 3.8, Aerospike re-architected its XDR component and merged the previously separate asxdr daemon into the main asd process. In addition, shipping performance is dramatically improved through the use of pipelined shipping. Please refer to the [3.8 release details page](/docs/operations/upgrade/xdr_to_3_8/release_details_3_8) for the full list of features and enhancements introduced in XDR as part of Aerospike 3.8.

This page describes the required steps to upgrade from a previous version of XDR to version 3.8.

{{#note}}
For one way shipping topologies, the following steps can be executed first at the source cluster or at the destintation cluster(s).
It is good practice, though, to first upgrade the destination cluster(s). As some of the statistic names have changed, the AMC package
should be upgraded to version 3.6.8.1 or above. Please contact [Aerospike Support](http://support.aerospike.com) 
if you need clarification on any of the following steps.
{{/note}}


{{#steps}}

{{#steps-step 1 "Preparation steps" markdown=true}}
The following points are optional but recommended for most production setups.

1. Verify that the cluster isn't migrating data. If it is migrating, allow the cluster to finish before
   proceeding. Use the following command to verify migrations are complete:
   ```bash
   asadm -e "asinfo -v statistics -l " | grep migrate_progress
   ```
   Migrations are complete when all `migrate_progress_recv` and `migrate_progress_send` values are 0.

2. If there are no active migrations, backup the cluster. For each namespace run:

    ```bash
    asbackup --host <nodeIP> --directory <path/to/backup/namespace> --namespace <namespace-name>
    ```

3. Save the current configuration file on each node. Assuming default location:

    ```bash
    sudo cp /etc/aerospike/aerospike.conf /etc/aerospike/aerospike.conf.bac
    ```

{{/steps-step}}


{{#steps-step 2 "Update the configuration file on each node" markdown=true}}
A few of the previous required configuration parameters are not necessary anymore:

1. Having the xdr daemon part of the asd process obsoletes the following configuration parameters:

  `xdr-namedpipe-path`  - not required anymore, xdr in same process as asd.<br>
  `xdr-local-node-port` - not required anymore, xdr in same process as asd.<br>
  `xdr-pidfile`         - not required anymore, xdr in same process as asd.<br>
  `xdr-errorlog-path`   - see point 3. below.<br>
  `xdr-info-port`       - can still be used, see point 4. below.<br>
  `stop-writes-noxdr`   - not required anymore, xdr in same process as asd.<br>

2. The following configuration parameters have been deprecated:

  `xdr-check-data-before-delete` - always on and built in version 3.8.<br>
  `xdr-read-batch-size`          - digests picked up and dispatched to specific threads (xdr-read-threads).<br>
  `xdr-hotkey-maxskip`           - new xdr-hotkey-time-ms introduced to control hotkey updates interval.<br>
  `xdr-max-recs-inflight`        - new tps based throttling through xdr-max-ship-throughput.<br>
  `xdr-forward-with-gencheck`    - should not be used as xdr generation check will break on delete/create<br>

  Contact Aerospike Support if you had any of the above configuration parameters defined in your configuration file.

3. By default, the XDR log messages are now merged with the main log file. If desired, those can still be separated, by updating the 
logging configuration as follows:

  ```
  logging {
    file /var/log/aerospike/aerospike.log {
      context any info 
      context xdr critical 
      context cf:rbuffer critical
    }

    file /var/log/aerospike/asxdr.log {
      context any critical 
      context xdr info
      context cf:rbuffer info
    }
  }
  ```

4. The XDR configuration and statistics are now accessible through the same port as the asd process (default 3000). For backward compatibility with potential monitoring and management tools, the `xdr-info-port` can still be configured and will return the same results as the asd info port.

5. The `xdr-digestlog-path` (which used to be `digestlog-path`) and `enable-xdr` are the only configuration parameters that are mandatory
in the xdr stanza, along with the datacenter substanza(s). Here is a simple example of a basic xdr stanza:

  ```
  xdr { 
      enable-xdr true 
      xdr-digestlog-path /opt/aerospike/xdr/digestlog 100G 

      datacenter DC1 {
        dc-node-address-port xx.xx.xx.xx 3000
        dc-node-address-port yy.yy.yy.yy 3000
        dc-node-address-port zz.zz.zz.zz 3000
      }
  } 
  ```

6. In lieu of `dc-int-ext-ipmap` configuration, which needed to be done on every source cluster, we introduced `alternate-address`, 
  which is configured on each of the destination node.<br>
  **Destination Node Config:**

  ```
  network {
      service {
          address any
          port 3000
          access-address aa.bb.cc.dd
          alternate-address xx.xx.xx.xx
      }
  }
  ```

  If `alternate-address` is set, the source cluster has to specify `dc-use-alternate-services true` in order to use the alternate IP address 
  to connect to the node (instead of services).<br>
  **Source Node Config:**

  ```
  xdr {
  ...
      datacenter DC1 {
          dc-node-address-port xx.xx.xx.xx 3000
          ...
          dc-use-alternate-services true
      }
  }
  ```

7. Other compatibility call-outs:

  - Local reads before shipping are always internal single-record reads in this new version. Therefore, the xdr initiated reads will not
  show up as read transactions. The xdr initiated read transactions can be tracked through the `stat_read_reqs_xdr` metric.

  - The "noresume" and "resume-nofailover" functionality is obsolete on asd service restart. By default, xdr will resume with failover.

For more information about latest configuration parameters, refer to the [Configuration Reference](/docs/reference/configuration).
{{/steps-step}}

{{#note}}
Execute steps 3 to 6 on each node, waiting for the node to join the cluster, and, depending on your requirements, waiting for migrations
to complete, before proceeding to the next node.
{{/note}}

{{#steps-step 3 "Stop the asd and asxdr daemons" markdown=true}}

```bash
$ sudo service aerospike stop 
Stopping aerospike:                                        [  OK  ]
$ sudo service aerospike_xdr stop
Stopping aerospike XDR:                                    [  OK  ]
```

Verify that the services are stopped:

```bash
$ sudo service aerospike status
asd is stopped
$ sudo service aerospike_xdr status
asxdr is stopped
```

{{/steps-step}}

{{#steps-step 4 "Install Aerospike version 3.8" markdown=true}}
Locate the asinstall script in the extracted media. Run the following command to upgrade the node:
```bash
$ sudo ./asinstall
```

{{/steps-step}}

{{#steps-step 5 "Start the new asd daemon" markdown=true}}
Start Aerospike and verify that it starts:
```bash
$ sudo service aerospike start 
Starting and checking aerospike:                       [  OK  ]
$ sudo service aerospike status
asd (pid 4389) is running...
```

{{#warn}}
During the rolling upgrade, following messages may be seen in the logs of nodes yet to be upgraded:<br>
{{/warn}}

`WARNING (xdr): (xdr_serverside.c::134) Got an unknown XDR message`<br> 
`WARNING (xdr): (xdr_serverside.c::134) Got an unknown XDR message`<br>
`WARNING (xdr): (xdr_serverside.c::134) Got an unknown XDR message`<br> 
`WARNING (xdr): (xdr_serverside.c::134) Got an unknown XDR message`<br>


{{#warn}}
Those messages would appear during migrations if there was a bit of lag. They indicate that a node running the upgraded version is trying to relog a digest against a node that has not been upgraded yet. During migrations, master partitions ownership would change, requiring digests to be relogged against the new owner of the partition. This type of relogging messages is not supported in previous XDR versions. This message should disappear once migrations complete and the digest corresponding to the migrations timeframe have been processed.
{{/warn}}

{{/steps-step}}


{{#steps-step 6 "Validate the installation" markdown=true}}
Verify in the logs that after restart each node is connecting to the correct destination and that it is shipping records (if there are records to be shipped). Check the logs for any unexpected warning messages.

{{#note}}
Contact Aerospike Support for any questions or issues encountered during this upgrade process.
{{/note}}
{{/steps-step}}
{{/steps}}

