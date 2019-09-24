---
title: Recommendations for Google Compute Engine
description: |
  Follow these easy steps to achieve a successful deployment of the Aerospike database on Google Compute Engine
scripts:
  - /assets/scripts/utils/target_blank.js

---

### OS

We recommend using the latest [Ubuntu 18](https://console.cloud.google.com/marketplace/browse?q=ubuntu&filter=category:os) OS as it has the most recent optimizations and bug fixes on the Google Compute Engine platform.
{{#tbd}}
You may also choose [Centos7](https://cloud.google.com/compute/docs/operating-systems/linux-os#centos7) or [RHEL](https://cloud.google.com/compute/docs/operating-systems/linux-os#red_hat_enterprise_linux_rhel) to run Aerospike.
{{/tbd}}

{{md (path-resolve (path-dirname page.src) "../../../operations/configure/network/heartbeat/cloud_consideration.part.md") }}

### Network Setup

By default, machines within a Google Compute Engine project can communicate freely with each other. A **default-allow-internal** firewall rule allows this. Aerospike uses TCP ports **3000** for communication with clients and **3002-3003** for intra-cluster communication. These ports need not be open to the rest of internet. If the database clients are present within the same project, there should not be any need for a separate firewall rule, as they will be able to connect over port 3000.

The [Aerospike Management Console (AMC)](/docs/amc) requires port **8081** for access from the internet.

You will need a port for **SSH access** to your instances (default tcp port **22**). This is also already open due to the **default-allow-ssh** firewall rule.

All instances in Google Compute Engine are assigned an internal IP address. These internal IP addresses should be used in the [mesh heartbeat](/docs/operations/configure/network/heartbeat/index.html#mesh-unicast-heartbeat) configuration.

### Persistence

#### Persistent Disk Storage

Google Compute Engine provides storage in the form of [Persistent Disks](https://cloud.google.com/compute/docs/disks), which are network-attached to virtual machine instances. There are two types of Persistent Disks: Standard (HDD) Persistent Disks and Solid-State Drive (SSD) Persistent Disks.
The performance of Persistent Disks is closely tied to the size of the disk volumes and the virtual machine instance types.

The Google Compute Engine [documentation](https://cloud.google.com/compute/docs/disks#typeofdisks) states:
  * IOPS performance limits grow linearly with the size of the persistent disk volume.
  * Throughput limits also grow linearly, up to the maximum bandwidth for the virtual machine that the persistent disk is attached to.
  * Larger virtual machines have higher bandwidth limits than smaller virtual machines.

We recommend using the SSD Persistent Disks instead of standard (HDD) Persistent Disks for the storage engines requiring persistence. Read more about [configuring storage engines](/docs/operations/configure/namespace/storage).

#### Local SSD Storage

In some zones, Google Compute Engine offers [local SSD storage](https://cloud.google.com/compute/docs/local-ssd) option. This provides extremely good performance, with high input/output operations per second (IOPS) and low latency compared to the standard persistent disk and SSD persistent disk options. However, these local SSDs are created and destroyed along with the virtual machine instance. In spite of this, the local SSD storage option can be used judiciously with an Aerospike cluster so that the data is always replicated on multiple local SSDs attached to multiple virtual machines in the cluster. 

Here is an example configuration snippet:
```
    storage-engine device {
            device /dev/disk/by-id/google-local-disk-0
			...
    }
```

It does not matter which interface type is chosen, scsi or NVMe. 

Ubuntu based OS deployments are already NVMe optimized. If deployed through the marketplace, the base image is scsi-mq optimized as well.

#### Shadow Device configuration

{{#info}}
Aerospike recommends Shadow Device configuration for cloud based VM deployments like in GCE.
{{/info}}

To take advantage of local disks with the persistence guarantee of persistent disks, Aerospike has the [Shadow Device configuration](/docs/operations/configure/namespace/storage#recipe-for-shadow-device) for the storage engine.

Note that the write throughput will still be limited by the instance limit and storage volume and hence this strategy will give good results when the percentage of writes is low.

An example config would be as follows:
```
    storage-engine device{
        device /dev/disk/by-id/google-local-ssd-0 /dev/disk/by-id/google-persistent-disk-1
        ...
    }
```
or using the resolved symlinks:
```
    storage-engine device{
        device /dev/nvme0n1 /dev/sdb
        ...
    }
```

{{#note}}
The shadow device should have a size at least equal to the size of the direct-attached SSD.  
{{/note}}
{{#note}}
For data-in-memory use cases with persistence, it may also be preferable to use a local SSD device alongside a Persistent Disk volume. In this case, it would be to save on IOPS cost incurred on read during the defragmentation process. The reads would be performed against the local SSD device and re-written/defragmented blocks directly mirrored to the Persistent Disk volume.
{{/note}}

