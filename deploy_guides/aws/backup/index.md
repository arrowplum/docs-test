---
title: Backup and Recovery in AWS
description: |
  Here are recommendations for backup and recovery within AWS
---


AWS provides alternative methods for backing up your data. This document will detail the general methodology and the options available.

## Aerospike Tools

The simplest method, and most traditional. 

### Backup:

1. Take a normal backup via `asbackup`
2. Move the file to a safe location

### Restore:

Upload the backup file to a system with `asrestore`. Use `asrestore` to restore the data back into a cluster.

`cat BACKUP.GZ | gunzip | asrestore [...] --input-file -`

**Pros**:
* Simplest 
* Cheapest
* Contains SMD data

**Cons**:
* Long duration
* Additional server load while running
* All or nothing across the cluster

## EBS snapshots

If you are using persistence, either through using EBS directly, or via [shadow device(s)](/docs/deploy_guides/aws/recommendations/#shadow-device-configuration), you can take advantage of EBS snapshots. EBS snapshots are point-in-time backups of data. This is done at the data block level, transparant to the application (Aerospike).

### Backup:

Simply initiate snapshots on your EBS data volumes.

{{#warn}}
EBS snapshots are consistent within a volume, but may not be consistent between volumes. As such, writing to a particular system needs to be stopped entirely, if said system utilizes *multiple EBS volumes per namespace*. This is due to the (possible) delay between the snapshot events.
{{/warn}}
{{#warn}}
Snapshots do not contain [SMD](https://www.aerospike.com/docs/architecture/secondary-index.html#index-management). SMD lives under /opt/aerospike/smd and needs to be separately accounted for in a full cluster restore. Individual node restores are unaffected due to SMD being repopulated via other nodes.
{{/warn}}

{{#note}}
Snapshots are taken quickly, but that does not mean it is immediately available for use. It will take some time before a snapshot is ready to be used.

Since snapshots are incremental, the first snapshot of any volume will also be the slowest to be ready.
{{/note}}


### Restore:

If your backups were taken via EBS snapshots, you would need to deploy said snapshots into EBS Volumes. Once you have your replacement EBS volumes provisioned, reconfigure the new replacement instance exactly as how the failed instance was configured.

This means that EBS volumes are mounted at the same locations and the same number of ephemeral drives are provisioned. 

{{#info}}
Shadow devices are always treated as authoritative upon cold-start. That means no matter what data may be on the local device, data will always be synchronized from the shadow device back into the local device.
{{/info}}

{{#warn}}
Be careful when restoring anything short of a full cluster. Due to how Aerospike manages data, there is a risk of records coming back. 
{{/warn}}


**Pros**:
* Quick
* Minimal impact
* Individual node based

**Cons**:
* More costly
* Additional scripting/coordination required
* Does not account for SMD


## Instance Failure

Suppose your instance failed, but you were using EBS for persistence. In this scenario, you can detach the EBS volume from the failed instance, and reattach it to a replacement instance. 

1. Remove/Terminate the failed instance, but keep the EBS volume.
2. Deploy a new Instance, with the same private IP as the old instance in the same zone.
3. Attach EBS volume from Step 1 to the replacement instance.
4. Restore the config to the replacement node and restart the node.

Once the node rejoins the cluster, migrations will occur and data will be rebalanced.
