---
title: Special Upgrades
description: Some versions require special instruction for upgrading 
styles:
  - /assets/styles/ui/steps.css
---

Below is the list of different versions requiring special instructions for
upgrading.

## Upgrading from pre-4.5.1 - SMD protocol change
- If running version prior to 4.5.1, please read
  [**4.5.1+ SMD protocol change**](/docs/operations/upgrade/aerospike/special_upgrades/4.5.1/).

## Upgrading to 4.5.1 or above
- In the unlikely event of having to perform a downgrade to Aerospike Server version 4.3.1.5 or earlier (or 4.4.0.4), the System Meta Data (SMD) content needs to be manually restored from the earlier running version. Downgrading in place will cause  all SMD content to be lost. This includes: Secondary index, UDF, security, truncate and eviction entries. Therefore, **prior to upgrading** to Aerospike Server version 4.5.1 or above, from an earlier version, backup the System MetaData (SMD) files. The default location for the System MetaData files is /opt/aerospike/smd. If no backup exists the SMD definitions will need to be rebuilt when downgrading.

## Upgrading to 4.3 with rack-aware and replication factor 2 or more

- If running with rack-aware and replication factor 2 or more, refer to the
  [**Special Considerations for Upgrading to 4.3**](https://discuss.aerospike.com/t/special-considerations-for-upgrading-to-version-4-3-rf-2-rack-aware-ap/5490)
  article.

## Upgrading to or over 4.2

- For clusters being upgraded from a version prior to **4.2** to **4.2** or
  later, storage devices and files must be deleted due to a change in storage
  format. Refer to the [**4.2 special upgrade steps**](/docs/operations/upgrade/storage_to_4_2/)
  for further details.

## Upgrading over 3.13

- For clusters being upgraded from a version prior to **3.13** to **3.13** or
  later, it should be noted that **3.13** is a required step-version upgrade in
  order to upgrade switch to the new cluster protocol. Refer to the
  [**3.13 special upgrade steps**](/docs/operations/upgrade/cluster_to_3_13/)
  for further details.
- For clusters running the cluster protocol introduced in version **3.13**, it
  is not necessary to wait for migrations to complete prior to proceeding to the
  next node. Simply wait for the upgraded node to re-join the cluster and
  proceed to upgrading the next one. The exception being for nodes restarted
  empty.
- For clusters not running the updated clustering protocol introduced in version
  **3.13**, it may be advisable to wait for migrations to complete after a node
  has been upgraded and prior to taking the next node down. Enterprise Edition
  licensees should contact Aerospike Support for further details.

## Upgrading to or over 3.11

- Clusters being upgraded passed version **3.11**, it should be noted that the
  restart of the service will trigger a [cold-start](/docs/operations/manage/aerospike/cold_start).
  It is recommended to review the documentation regarding [cold-start](/docs/operations/manage/aerospike/cold_start).

## Upgrading to or over 3.10

- For clusters being upgraded passed version **3.10**, the client libraries need
  to be running the following versions or above:
  - C: 3.0.26 (September 11, 2013)
  - C#/.NET: 3.1.2 (April 27, 2015)
  - C Libevent: 2.1.20 (September 24, 2013)
  - Go: 1.0 (no upgrade required)
  - Java: 3.1.1 (April 15, 2015)
  - Node.js: 1.0.27 (January 29, 2015)
  - Python: 1.0.35 (January 8, 2015)
  - Ruby: 1.0 (no upgrade required)
- For clusters being upgraded passed version **3.10**, review the
  [network configuration migration guide](/docs/operations/upgrade/network_to_3_10/).
- For clusters being upgraded passed version **3.9**, note that
  [the statistics naming and structure](/docs/operations/upgrade/stats_to_3_9/)
  was overhauled. In general, it is advised to review the monitoring dashboards
  in a QA environment prior to making upgrades in production to avoid any
  unexpected changes.
