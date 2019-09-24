---
title: Stats and Benchmark Migration Guide for the 3.9 Release
description: Understanding statistics and benchmark changes in 3.9.
styles:
  - /assets/styles/ui/steps.css
---

In version 3.9, Aerospike has revamped its statistics and benchmarking capability. The change are driven by the following requests from our users -
* Ability to track namespace-level activity, for both statistics and benchmarking.
* Better delineation of different usages (read/write/batch etc).
* More statistics tracking in server log.

This document provides change details which will help you migrate your existing monitoring setup, and making future enhancement to take advantage of the new statistics structure.

Please refer to [Metric Reference](/docs/reference/metrics) for detailed list of deprecated, renamed, and added statistics.

For benchmark differences, please refer to [Latency](/docs/operations/monitor/latency).

For log line differences, please refer to [Log Messages](/docs/reference/serverlogmessages).

## Statistics Change

The biggest change is moving as many statistics as possible to be tracked at the namespace level. This includes -
* Single-record operation usage counters such as client read, write, delete, or udf.
* Multi-record operation usage counters such as scan, query
* Internally generated single-record operation usage counters such as xdr read, batch read, scan background udf.
* Data Rebalance (Migration) progress counters.
* Memory/Device usage counters.

For operation statistics, the counter is incremented on operation completion, not on attempts, and thus do not overlap. For example, an in-coming client read request will be accounted in one of three counters - client_read_success, client_read_error, or client_read_timeout.
 
Some of the existing namespace statistics are renamed to achieve naming consistency -
* All statistics have underscore (_) in between words.
* The name has most differentiating component at the beginning. 

For example, `data-used-bytes-memory` is now named `memory_used_data_bytes`.

### Statistics Aggregation

A few statistics remain as aggregations over the namespaces. This include -
* migration - migrate_partitions_remaining, migrate_partitions_remaining.
* objects & sub_objects.

### Use wildcards for high level statistics

A number of high level statistics in common use have been deprecated, such as `stat_read_req` and `stat_write_req`.

Instead of Aerospike internally supporting a wide variety of aggregated statistics, we have moved to supporting fine grained statistics, and provide the simple ability for you to aggreate within your monitoring tool.

The following wildcard patterns are supported with the following meanings. Where `[ns]` is notated, you can either use
an explicit namespace, or use your wildcard pattern to achieve the global statistic.

* [ns].client\_\*\_error - All errors ( read, write, delete, proxy ) returned to clients.
* [ns].client\_\*\_success - All successful transactions ( read, write, delete, proxy ).
* [ns].client\_\*\_timeout - All transactions which have timed out
* [ns].client\_read\_\* - All read requests ( which does not include scan and query traffic )
* [ns].client\_write\_\* - All write requests
* [ns].client\_delete\_\* - All delete requests
* [ns].client\_udf\_\* - All single record UDF requests

In specific, two common monitoring metrics become available on the namespace.

* `stat_read_req` becomes \*.client\_read\_\* 
* `stat_write_req` becomes \*.client\_write\_\* 

## Tools Compatibility

The following tools versions are required to work with 3.9 server statistics. All tools listed below maintain the same level of backward compatibilty with previous version of Aerospike Server. You will still need to determine if your statistics have been deprecated and whether to shift your monitoring to a different statistic.

* Aerospike Monitoring Console (AMC) - require AMC 3.6.9
* Aerospike Admin Command Tool (asadm) - require asadm 0.1.4
* Aerospike Collectd plug-in (collectd) - require 1.0.3
* Aerospike Nagios plug-in - recommend 1.3.1 for unit of measurement on new metrics
* Aerospike Graphite plug-in - recommend 1.5.3 for easier query forming on new latency metrics 
* Aerospike Zabbix plug-in - no update required. UI require updating to new stats.

## Key Metrics Changes

Aerospike recommends a handful of [key metrics](/docs/operations/monitor/key_metrics) to monitor. 

Only minor renaming changes have been made for these key metrics.

* hwm-breached is now hwm_breached
* stop-writes is now stop_writes

## Upgrade Process
To have the smoothest upgrade, it is recommended to -

* Upgrade your tool usages - AMC, asadm, or any third-party plug-in you use. These tools will be backward compatible with both older and newer versions of Aerospike.
* Change your UI pane to accomodate the new statistics. 
* If you are using specific scripts for monitoring out side of AMC or collectd, make adjustments of your scripts based on the new statistics.
* Then upgrade your server.


## Other Notable Changes
A few other operational changes in 3.9 are also described below -

### Get-config Response Scoping
A common way to get server configuration is through the 'asinfo -v get-config' call. 
The output format of this call is enhanced so that subscoped sections will specify the scope name, with a dot (.) as a delimitor.
For example, the namespace storage configuration `data-in-memory=true` will now be returned as `storage-engine.data-in-memory=true`


### Deprecated Tools
Also in 3.9, we have deprecated some of our older generation tools.
* asmonitor - replaced by asadm
* cli - absorbed by aql
* ascli - absorbed by aql
* AscollectInfo - absorbed by asadm
