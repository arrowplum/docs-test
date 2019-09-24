---
title: Monitoring
description: Monitoring the Aerospike database
---

Monitoring is a critical component for any mission critical deployment. The
obvious reason to monitor is to decrease operational response time to outage
events such as hardware failure and software errors. Most monitoring tools
(such as [Graphite](http://www.graphiteapp.org/) or
[Nagios](http://www.nagios.org/)) can provide trend data to allow your operations
team to effectively recognize and address future scale hurdles.

### Aerospike Metrics

Aerospike offers a extensive suite of [Metrics](/docs/reference/metrics) that
you may use for your monitoring solution. See [Key Metrics](/docs/operations/monitor/key_metrics) for a
list of recommended metrics to monitor to help you get started with your
monitoirng solution.

### Monitoring Solutions

Aerospike has several different methods for monitoring the database including
standalone tools and integration plugins for 3<sup>rd</sup> party plugins.

| Tool                                      | Documentation                             | Alerting       | Trending       |
|-------------------------------------------|-------------------------------------------|----------------|----------------|
| [Aerospike Monitoring Console](/docs/amc) | [Aerospike Monitoring Console](/docs/amc) | Yes            | No             |
| [ASADM](/docs/tools/asadm)                | [ASADM](/docs/tools/asadm)                | No             | No             |
| [Aerospike Logs](/docs/operations/monitor/latency)                 | [Aerospike Logs](/docs/operations/monitor/latency)                 | No             | Yes            |
| [Collectd](https://www.collectd.org)      | [Guide](/docs/operations/monitor/collectd)                         | Yes<sup>*</sup>| Yes<sup>*</sup>|
| [Graphite](http://graphite.wikidot.com/)  | [Guide](/docs/operations/monitor/graphite)                         | No<sup>*</sup> | Yes            |
| [Nagios](http://www.nagios.org)           | [Guide](/docs/operations/monitor/nagios)                           | Yes            | No<sup>*</sup> |
| [Zabbix](http://www.zabbix.com)           | [Guide](/docs/operations/monitor/zabbix)                           | Yes            | Yes            |
<sup>* Solution has 3rd party plugin for Alerting or Trending</sup>

{{#todo}}
Remove the following link.
[.](/docs/operations/monitor/interactive)
{{/todo}}
