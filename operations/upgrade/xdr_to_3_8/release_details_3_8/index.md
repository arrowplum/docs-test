---
title: Release Details - XDR in Aerospike 3.8
description: Details of the Aerospike 3.8 release in term of new features, compatibility call outs and other changes in XDR.
styles:
  - /assets/styles/ui/steps.css
---

This page covers changes to the XDR component, introduced as part of Aerospike version 3.8.

## New Features

There are two major changes in the 3.8 XDR component.
- The XDR process (asxdr) is now integrated in the main Aerospike Database process (asd). Previously XDR is a separate daemon, which needed separate operational management.
- The latest Aerospike C client is incorporated as the shipping client.

As a result, the following features and enhancements are introduced.

1. **Pipelined Shipping** - The new in-process XDR uses transaction pipelining. As a result, multiple write transactions can be "in-flight" on the same established connection, resulting in much higher throughput over WAN links while using a much lower number of open connections.

2. **Security** - For destination datacenters that require ACL security, the in-process XDR now supports ACL security. Different security user and credentials can be used by the source datacenter to ship to different destination data centers. Details on how to configure security for XDR can be found on the [cross-datacenter configuration](/docs/operations/configure/cross-datacenter) page.

## Enhancements

1. **Throughput Based Throttling** - introduced new dynamic configuration `xdr-max-ship-throughput`. This configuration controls the speed at which source data center ships. It replaces `xdr-max-recs-inflight`. This is on a per DC basis. So if shipping to multiple DC, the xdr-max-ship-throughput will be applied for each DC individually.

2. **Time Based Hot-key Management** - introduced new dynamic configuration `xdr-hotkey-time-ms`. This configuration controls how much time (in milliseconds) to wait in between shipping of hot-keys.  It replaces `xdr-hotkey-maxskip`.

3. **Key Shipping** - When keys are stored for records, XDR now ships them.

4. **Simpler IP Mapping (Alternate)** - introduced new static configuration `alternate-address`. In lieu of dc-int-ext-ipmap configuration, which needed to be done on every source cluster, the `alternate-address` is configured on each of the destination node, to indicate what IP address the source datacenter should use to connect. Details on how to configure 
`alternate-address` for XDR can be found on the [cross-datacenter configuration](/docs/operations/configure/cross-datacenter) page.

5. **Combined Stats** - XDR statistics are returned as part of the overall statistics from the Aerospike Database (default port of 3000).
 `asinfo -v "statistics/xdr"` will provide XDR specific statistics from the asd port (default 3000). If `xdr-info-port` is 
 configured, it will give the same output as the asd port.

6. **Combined Logs** - By default, the XDR log messages are now merged with the main log file. If desired, those can still be separated.
See [Upgrading XDR to 3.8](/docs/operations/upgrade/xdr_to_3_8) for details.

7. **Dynamic DC configuration** - Seeding of datacenters can be done dynamically. See details on the 
[cross-datacenter configuration](/docs/operations/configure/cross-datacenter) page. It is recommended to always 
configure datacenters in the configuration and restart asd to make sure the configuration is correct upon subsequent restart of a node.

## Compatibility call outs

1. **Deprecated configuration variables** - Having the xdr daemon part of the asd process obsoletes the following configuration parameters:

  `xdr-namedpipe-path`  - not required anymore, xdr in same process as asd.<br>
  `xdr-local-node-port` - not required anymore, xdr in same process as asd.<br>
  `xdr-pidfile`         - not required anymore, xdr in same process as asd.<br>
  `xdr-errorlog-path`   - logs now unified with asd by default.<br>
  `xdr-info-port`       - can still be used for backward tools compatibility.<br>
  `stop-writes-noxdr`   - not required anymore, xdr in same process as asd.<br>
  `xdr-check-data-before-delete` - always on and built in version 3.8.<br>
  `xdr-read-batch-size`          - digests picked up and dispatched to specific threads (`xdr-read-threads`).<br>
  `xdr-hotkey-maxskip`           - new `xdr-hotkey-time-ms` introduced to control hotkey updates interval.<br>
  `xdr-max-recs-inflight`        - new tps based throttling through `xdr-max-ship-throughput`.<br>
  `xdr-forward-with-gencheck`    - should not be used as xdr generation check will break on delete/create<br>

  Contact Aerospike Support if you had any of the above configuration parameters defined in your configuration file.

2. **Local reads** - Local reads before shipping are always internal single-record reads in this new version. Therefore, the xdr initiated reads will not
  show up as read transactions. The xdr initiated read transactions can be tracked through the `stat_read_reqs_xdr` metric.

3. **Resume/Nofailover** - The "noresume" and "resume-nofailover" functionality is obsolete on asd service restart. By default, xdr will resume with failover.


For more information about latest configuration parameters, refer to the [Configuration Reference](/docs/reference/configuration).
