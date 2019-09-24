---
title: Recommendations for Microsoft Azure
description: |
  Follow these easy steps to achieve a successful deployment of the Aerospike database on Microsoft Azure
scripts:
  - /assets/scripts/utils/target_blank.js

---


### OS

We recommend using the latest Ubuntu Server LTS as it has the most recent optimizations and bug fixes on Azure.
You can find Ubuntu through the Azure portal or at [this page](https://azuremarketplace.microsoft.com/marketplace/apps/Canonical.UbuntuServer).

### Instance Type

**Lsv2** family of VMs are specifically designed for high performance IO applications and is the primary recommendation of Aerospike.

However, the **M** and **DSv2** can also be utilized should their characterisitcs match your usage.

In general, any persistence based storage should utilze VM families with the 'S' suffix. This indicates the VMs have Premium (SSD) Storage support.

{{#note}}
Aerospike recommends enabling [Write Acceleration](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/how-to-enable-write-accelerator).
{{/note}}

### Network Setup

By default, machines within the same Virtual Network in Azure can communicate freely with each other. Aerospike uses TCP ports **3000** for communication with clients and **3002-3003** for intra-cluster communication. These ports need not be open to the rest of internet. If the database clients are present within the same Virtual Network, there should not be any need for a separate firewall rule, as they will be able to connect over port 3000.

The [Aerospike Management Console (AMC)](/docs/amc) requires port **8081** for access from the internet.

You will need a port for **SSH access** to your instances (default tcp port **22**). 

All instances in Azure are assigned an internal IP address. These internal IP addresses should be used in the [mesh heartbeat](/docs/operations/configure/network/heartbeat/index.html#mesh-unicast-heartbeat) configuration.

### Persistent Disks

Azure provides storage in the form of VHD Disks in [Storage Accounts](https://docs.microsoft.com/en-us/azure/storage/storage-introduction). 

There are two types of disks. High performance Premium Storage offers SSD backed storage, while Standard Storage offers HDD backed storage.
The performance of the disk is closely tied to the size of the disk volume.

{{#note}}
As of this writing, the [best performant](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/premium-storage#premium-storage-disk-limits)
 disk is the P40 (2048GB) disk. There is a bigger P50 (4096GB) disk, but it does not confer additional performance benefits.
{{/note}}
{{#note}}
Azure [limits](https://docs.microsoft.com/en-us/azure/storage/storage-premium-storage#scalability-and-performance-targets) individual persistent disk performance. To achieve higher performance, additional disks need to be provisioned and used in parallel.
{{/note}}

### Local SSD

Some Instance types come with local SSDs. This provides extremely good performance, with high input/output operations per second (IOPS) and low latency compared to the persistent disk options. However, these local SSDs are created and destroyed along with the virtual machine instance. In spite of this, the local SSD storage option can be used judiciously with an Aerospike cluster so that the data is always replicated on multiple local SSDs attached to multiple virtual machines in the cluster. 

Here is an example configuration snippet:
```
    storage-engine device {
            device /dev/sdb
    }
```

Local SSD IOPS **are not limited** by the persistent Disk IOPS allocations, nor instance IOPS allocations. They have their own allocations.

{{#info}}
See [Premium Storage scalability](https://docs.microsoft.com/en-us/azure/storage/storage-premium-storage#premium-storage-scalability-and-performance-targets).
{{/info}}

## Shadow Device configuration

As noted above, some Azure instance types have local SSDs. These can be significantly faster than Premium Storage Disks, as they are network attached. But Azure treats local SSDs as cache and not suitable for long-term data, as these volumes are purged when the instance stops.

To take advantage of local disks with the persistence guarantee of Azure Blob Storage, Aerospike has the [Shadow Device configuration](/docs/operations/configure/namespace/storage#recipe-for-shadow-device) for the storage engine.

Note that the write throughput will still be limited by the instance limit and storage volume and hence this strategy will give good results when the percentage of writes is low.

An example config would be as follows:
```
	storage-engine device{
		device /dev/sdb	/dev/sdc
		...
	}
```

{{#note}}
The shadow device should have a size at least equal to the size of the direct-attached SSD.  
1023GB (the maximum as of this writing) should always be used in Azure.
{{/note}}
{{#note}}
For data-in-memory use cases with persistence, it may also be preferable to use a local SSD device alongside a Premium Storage Disk volume. In this case, it would be to save on IOPS cost incurred on read during the defragmentation process. The reads would be performed against the local SSD device and re-written/defragmented blocks directly mirrored to the Premium Storage Disk volume.
{{/note}}


### Fault Tolerance

Azure has the concept of [Availability Sets](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-manage-availability/). It consists of Update Domains and Fault Domains.

![Update Domains and Fault Domains](/docs/deploy_guides/azure/assets/azure-udfd.jpg)

Update Domains are groups of physical systems that can be rebooted simultaneously in a maintenance event.  
Fault Domains are groups of physical systems that share power and network switch.  

{{#note}}
Aerospike suggests having the Availability Set to be defined with the most number of Update Domains and Fault Domains as possible. 
{{/note}}
{{#info}}
Azure will not give notice of VM reboots for Update Domain interruptions.
{{/info}}
{{#warn}}
VMs are migrated by Azure automatically should a Fault Domain issue arise. This live-migration process has historically been detrimental to Aerospike. Aerospike can be made more resiliant against live migrations, but at the cost of performance.
{{/warn}}
{{#info}}
Aerospike also provides XDR replication in our Enterprise Edition to facilitate DR scenarios
{{/info}}


### Additional Information

* [Optimize your Linux VM on Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/optimization)
