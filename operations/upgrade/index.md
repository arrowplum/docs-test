---
title: Upgrade or Repair Server
description: Learn how to upgrade an individual Aerospike server without stopping the cluster without data loss or experiencing service downtime.
---

Aerospike supports upgrading a cluster without service downtime and without data
loss.  That is, you can "hot swap" individual servers by stopping a server,
upgrading it and restarting it, as described below.  Upgrading an individual
server does not stop the cluster from continuing to provide database access.

{{#warn}}
Refer to the [**Special Upgrade instructions**](/docs/operations/upgrade/aerospike/special_upgrades/index.html)
page for important upgrade information relevant to specific versions.
{{/warn}}

You can:
- [Upgrade Aerospike](/docs/operations/upgrade/aerospike): upgrade from one
  version to another.
- [Upgrade Hardware](/docs/operations/upgrade/hardware): upgrade with new
  hardware or replace failed hardware.

Before upgrading, review the [release notes](/download/server/notes.html) to see
the changes in the planned versions and any special considerations which may
have been noted.

## Estimating the Time Required for an Upgrade

Aerospike Database handles large quantities of data, so when servers must be
upgraded or restarted, it can take some period of time for the cluster to
re-synchronize, re-balance and return to its normal levels of performance. It is
difficult to estimate how much time an upgrade will take.

### Some situations to watch for:

- If you install a new SSD drive, refer to the
  [How to add, replace and remove disks](https://discuss.aerospike.com/t/how-to-add-replace-remove-disks/746)
  knowledge base article.
- If your configuration/situation allows for a
  [fast restart](/docs/operations/manage/aerospike/fast_start), the server
  restart typically only takes a few minutes. For example, if you want to
  install a new software version on your cluster, it should be a matter of a few
  minutes to take down a server, install new software and then do a fast
  restart.
- If a server is down for a long time (more than an hour), the re-balancing
  process (migrations) will take longer. Also, if running on a version prior to
  3.13 (or 3.13 and not running the latest cluster protocol version) it is
  recommended to wait for migrations to complete prior to taking any further
  Aerospike process down to avoid any data consistency issue. 
