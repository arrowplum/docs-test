---
title: System Overview
description: Aerospike is a fast, highly scalable and reliable database for real-time, big data applications.  
assets: /docs/architecture/assets
---

The mission of Aerospike Database is to be very fast, highly scalable, and extremely reliable for use in real-time big data applications. The Operations Manual explains how to create and maintain an Aerospike implementation - plan, install, configure, manage, monitor, tune and troubleshoot. This Introduction gives an overview of the content - to help you understand the various subsections of the Operations Manual and to help guide you to the right material.

### [Plan](/docs/operations/plan)

This section covers how to plan and select the best hardware configuration for your application.

- [Linux Capacity Planning](/docs/operations/plan/capacity) - calculate total storage, RAM and throughput hardware requirements
- [Amazon EC2 Capacity Planning](/docs/deploy_guides/aws/plan) - choose the right instance type for your use case
- [Google Cloud Compute Capacity Planning](/docs/deploy_guides/gcp/plan) - performance numbers factored with your application's memory requirements will help identify the right machine type.
- [Server Hardware](/docs/operations/plan/hardware) - determine what hardware to use
- [Flash Storage](/docs/operations/plan/ssd) - specialized considerations for taking advantage of flash storage
- [Network](/docs/operations/plan/network) - how Aerospike uses the network

### [Install](/docs/operations/install)

This sections describes how to install Aerospike on Amazon EC2, different Linux distributions, OS X, Windows and on several cloud providers.

- [Install on Linux](/docs/operations/install) - how to install on Red Hat, Ubuntu, Debian and other Linux distributions
- [Install on OS X](/docs/operations/install/vagrant/mac) - using a Vagrant managed virtual machine on OS X
- [Install on Windows](/docs/operations/install/vagrant/win) - using a Vagrant managed virtual machine on Windows
- [Install on Amazon EC2](/docs/deploy_guides/aws/install) - how to launch an Aerospike Amazon Linux AMI
- [Install on Google Cloud Compute](/docs/deploy_guides/gcp/install) - Launch your Aerospike cluster in seconds using Google Compute Engine's Click to Deploy
- [Install on Other Clouds](/docs/operations/install) - how to deploy on other cloud services

### [Configure](/docs/operations/configure)

In Aerospike there is a single configuration file on each database node which specifies parameters for network, namespace, log and datacenter replication. For a given namespace most of the information in the configuration files will be the same.

- [Amazon EC2](/docs/deploy_guides/aws/recommendations) - recommendations for configuring port, ip address, heartbeat mode, rack awareness and other parameters
- [Google Cloud Compute](/docs/deploy_guides/gcp/recommendations) - recommendations for configuring network, firewall, and clusters
- [Network](/docs/operations/configure/network) - configure port, ip address, heartbeat mode, rack awareness and other parameters
- [Namespace](/docs/operations/configure/namespace) - configure data storage location, data retention and data replication
- [Access Control](/docs/operations/configure/security/access-control) - configure Access Control for user, role, and privilege creation and maintenance
- [LDAP](/docs/operations/configure/security/ldap) - configure using an External Authentication system
- [Encryption at Rest](/docs/operations/configure/security/encryption-at-rest) - configure encrypting database record data on storage devices using symmetric AES-128 encryption 
- [Consistency](/docs/operations/configure/consistency) - configure namespace with "strong-consistency"
- [Log](/docs/operations/configure/log) - configure log location and logging level, and learn use of logrotate tool
- [Datacenter Replication](/docs/operations/configure/cross-datacenter) - establish and configure Cross Datacenter Replication (XDR) for Aerospike Enterprise Edition customers (set parameters, establish topology, configure network and specify data replication)
- [Non-Root](/docs/operations/configure/non_root) - set-up Aerospike to run as a non-root user

### [Manage](/docs/operations/manage)

Aerospike management functions include starting and stopping Aerospike and XDR services, adjusting data retention policies, and managing Aerospike features like indexes, queries, scans, and UDFs. 

- [Aerospike Daemon](/docs/operations/manage/aerospike) - control the Aerospike Daemon with the SysV init script
- [Aerospike systemd](/docs/operations/manage/aerospike/systemd) - control the Aerospike Daemon with systemd
- [Datacenter Replication](/docs/operations/manage/xdr) - control the XDR Daemon with the SysV init script
- [Consistency](/docs/operations/manage/consistency) - add and remove nodes in strong consistency namespaces, as well as how to detect and repair unavailability
- [Log Files](/docs/operations/manage/log) - working with the Log File
- [Storage Capacity](/docs/operations/manage/storage) - setting data eviction, time-to-live and defragmentation parameters
- [Migrations](/docs/operations/manage/migration) - understanding, managing and monitoring migrations
- [Sets](/docs/operations/manage/sets) - use asinfo to set and manage parameters for a set
- [Indexes](/docs/operations/manage/indexes) - using the aql and asinfo tools to create and manage secondary indexes
- [Queries](/docs/operations/manage/queries) - use asinfo to set and update parameters for queries across a cluster
- [Scans](/docs/operations/manage/scans) - set configuration parameters to manage scans
- [UDFs](/docs/operations/manage/udfs) - using aql and asinfo tools or a Java, C# or C client to manage UDFs

### [Upgrade](/docs/operations/upgrade)

Aerospike supports upgrading a cluster or repairing a server without service downtime and without data loss.

- [Aerospike](/docs/operations/upgrade/aerospike) - upgrade cluster software
- [Hardware](/docs/operations/upgrade/hardware) - upgrade cluster hardware
- [Special Upgrades](/docs/operations/upgrade/aerospike/special_upgrades) - special instructions for upgrading
- [4.5.1+ SMD protocol change](/docs/operations/upgrade/aerospike/special_upgrades/4.5.1) - upgrade process for SMD protocol change in version 4.5.1+
- [Storage Format Upgrade in 4.2](/docs/operations/upgrade/storage_to_4_2) - upgrade process for internal storage format change
- [FAQ - 4.2 upgrade](/docs/operations/upgrade/storage_to_4_2/faq) - Frequently Asked Questions regarding the 4.2 upgrade
- [Cluster Protocols Upgrade in 3.13](/docs/operations/upgrade/cluster_to_3_13) - upgrade process for full cluster protocols switch-over in version 3.13
- [FAQ - 3.13 upgrade](/docs/operations/upgrade/cluster_to_3_13/faq) - Frequently Asked Questions regarding the 3.13 upgrade
- [Network upgrade in 3.10](/docs/operations/upgrade/network_to_3_10) - upgrade path for your existing clusters, and points out the major features that these changes bring
- [Stats upgrade to 3.9](/docs/operations/upgrade/stats_to_3_9) - stats and benchmark migration guide for the 3.9 release
- [XDR to 3.8](/docs/operations/upgrade/xdr_to_3_8) - upgrade XDR to version 3.8
- [Aerospike 2 to 3](/docs/operations/upgrade/2_to_3) - upgrade from Aerospike 2 to Aerospike 3
- [Community to Enterprise](/docs/operations/upgrade/community2enterprise) - upgrade from Community to Enterprise

### [Monitor](/docs/operations/monitor)

It is important to monitor your Aerospike system in order to decrease operational response time to outage events such as hardware failure and software errors. Also, some monitoring tools (such as Graphite or Nagios) can provide trend data to allow your operations team to effectively recognize and address future scale hurdles. Important metrics can be gathered in the areas of applications, memory, networks, storage, services and trends.

- [Key Metrics](/docs/operations/monitor/key_metrics) - recommended metrics to use for monitoring and trending
- [Latency](/docs/operations/monitor/latency) - access latency trends from Aerospike Logs
- [Collectd](/docs/operations/monitor/collectd) - configure collectd plugin
- [Graphite](/docs/operations/monitor/graphite) - configure asgraphite plugin
- [Nagios](/docs/operations/monitor/nagios) - configure asnagios plugin
- [Zabbix](/docs/operations/monitor/zabbix) - configure zabbix plugin


### [Troubleshoot](/docs/operations/troubleshoot)

{{#note}}
Make sure to consult and search over the [Knowledge-Base](https://discuss.aerospike.com/c/knowledge-base) topics as a wide variety of issues and remediations 
are covered there.
{{/note}}

What to do, step-by-step, to diagnose system problems.  Also, specific points in the several areas listed below.

- [Install](/docs/operations/troubleshoot/install) - problems with installation
- [Startup](/docs/operations/troubleshoot/startup) - problems with: ASD daemon, file descriptors in log, defrag loop, network device replacement
- [Node](/docs/operations/troubleshoot/node) - adjusting eviction rate to avoid an out of memory (OOM) problem
- [Cluster](/docs/operations/troubleshoot/cluster) - cluster integrity fault; check for node down; Paxos/fabric health issues after network glitch or cluster size change
- [Client](/docs/operations/troubleshoot/client) - receiving server memory errors, KEY_BUSY code, PHP causing segfaults
- [Dynamic Config](/docs/operations/troubleshoot/dynamic_config) - using asinfo to dynamically change parameters, and a list of several common parameter settings
- [Misc](/docs/operations/troubleshoot/misc) - fire-forget feature, transaction-pending-limit, response to stack trace, "key field too big"

### Reference Manuals

- [Configuration Parameters](/docs/reference/configuration)
- [Info Commands](/docs/reference/info)
- [Metrics](/docs/reference/metrics)
- [Server Log Messages](/docs/reference/serverlogmessages)


