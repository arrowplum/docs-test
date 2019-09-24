---
title: Flash Storage
description: Learn how Aerospike has been optimized to take advantage of Flash storage and how you can properly over-provision your SSDs.
---

When Flash devices (SSDs) are used for storage, they are often the most important hardware consideration. 
Aerospike has optimized the database to take advantage of Flash storage.

## How Aerospike uses Flash storage

- Because flash devices have no moving parts, they are already perfect for random reads. This gives near-RAM latencies, but provides durability and persistence.

- In order to avoid creating uneven wear on a single part of the SSD, Aerospike does not update data in-place. Instead, it employs a copy-on-write. This wears Flash evenly and means that Aerospike customers can expect very good durability.

- All databases must at some point reclaim data. While Aerospike can be configured with a file system (and must be for the specific case of
[flash index](/docs/operations/configure/namespace/index/index.html#flash-index)), Aerospike prefers to be configured with block device storage for data devices. This allows the database to run a background job that is able to reclaim space continually.

- Aerospike distributes the data across Flash devices. As a result, use of RAID is discouraged. If SAS or SATA drives are being used through an HBA or SAN, a `JBOD` configuration is preferred, allowing Aerospike to distribute load across the devices.

- Flash devices may perform better when over-provisioned. This is the space given to the controller to help with garbage collection, wear-leveling, and bad block mapping. While manufacturers already take this into account, some drives may benefit from extra overprovisioning in high-write uses.

{{#note}}Flash devices sometimes require over-provisioning in excess of that provided by the manufacturer. While this is critcal in older drives ( pre-2015 ), more recently manufacturers have
properly configured drives.{{/note}} 

{{#note}}Using cloud-based Flash storage has extra considerations. While you can't choose a different Flash drive, you will need to determine the correct instance types, and number of instances,
which will be required for your use. You also need to consider the ephemeral nature of direct attach Flash in cloud deployments, and configure a persistant store through our 
[shadow device method](/docs/operations/configure/namespace/storage/index.html#recipe-for-shadow-device).{{/note}}

## Recommendations

- Determine three planning factors: your object size, your read/write ratio and your required latency. Use either our Flash
planning guide, or use Aerospike's benchmarking tool, to determine how many drives, and what type, to acquire.
 
It is very important to note a few things about our tests:

- Aerospike uses a tool we created called ACT (available at [GitHub](http://github.com/aerospike/act)). This tool generates traffic similar to what Aerospike sees in production.
- Be careful comparing the results from our ACT test tool to other benchmarks. Aerospike has specific ways that it handles data that are different from consumer use. So SSDs that have performed very well in our benchmarks may be middling in other benchmarks, but the ones that do well in the other benchmarks do not pass our tests.
- Use direct attach rather than RAID, if possible.
- If you are using RAID controllers, please make sure you have them in JBOD (Just a Bunch Of Disks) mode or as a series of RAID 0 arrays (one per disk). As mentioned above, using RAID at any level adds latency. Striping across Flash devices will increase latency and decrease throughput.
- Because Aersopike uses SSDs as raw devices for data, you do not need to defragment the drive separately. Aerospike handles that on a continual basis, but the software does require space to do this. In order to have the best performance, you will need to have the Flash device 50% free. Settings for keeping the Flash device usage below this are in the configuration file.
- If you are using a VM (Virtual Machine) Flash performance may be affected depending on the details of how your VM software interacts with the host. We recommend researching the best platforms for use. In the case of Amazon EC2, for example, we only support the Amazon Linux Distribution operating system.

Installing and configuring Flash devices is not exactly the same as using rotational drives.

Aerospike recommends that you use dedicated drives or storage devices. This increases predictability for performance of any database,
but Aerospike's load on storage is extreme, and can interfere with logging and other OS processes. 
When looking at purchasing hardware make sure the there is at least one other drive for the operating system.

How you install the Flash devices into the server is dependent on the vendor of the server and your connector. In 2018, Flash devices have migrated from SATA to NVMe, which
comes in two form factors: PCIe cards (Add In Cards, or AIC, usually Half Height Half Length HHHL), or 
newer 2.5" U.2 devices, which look like SATA drives and will be inserted into a chassis through the front panel.

In the case of SATA configuration, you will likely have a RAID controller, which should be configured in pass-through mode.
  
{{#note}}You should not format your Flash devices that are going to be used for data. Aerospike uses raw disk for optimal performance with data records. Devices that will be used for 
[flash indexes](/docs/operations/configure/namespace/index/index.html#flash-index) will need to be formatted, however.{{/note}}

A somewhat more complex option is to use a RAID controller that cannot be set into pass-through mode.  
Some RAID controllers work fine but others do not.  When you can't use pass-through, you must set up each SSD as its own RAID-0 drive.

For example, the Dell H310 RAID controller works best if you:

- Put the data Flash devices on a different controller than the drive containing the OS.
- Move rotational drives to a different RAID controller – this requires a board upgrade that you must specify when you purchase the controller.

Check the documentation for your RAID controller for details.

As a best practice, in older server models where drives had to be installed inside the server chasis, before closing the server, make sure AHCI and NCQ are turned ON in the BIOS or the RAID card for the Flash devices. The process for doing this varies by hardware – consult your RAID documentation for details.

## Before you Buy
Before buying any Flash device, you should review:

- [Certifying flash devices](http://github.com/aerospike/act)
- [Flash device Setup](/docs/operations/plan/ssd/ssd_setup.html)
- [Flash device Initialization](/docs/operations/plan/ssd/ssd_init.html)

## Additional Information
 - For help with flash/SSD issues and to post your own questions, see [SSD discussions in the Aerospike forums](https://discuss.aerospike.com)
 - For a list of tested SSDs, see [Flash Certification](/docs/operations/plan/ssd/ssd_certification.html)
&nbsp;

---

<div class="clearfix"></div>
<div class="pull-left">
<a class="btn btn-default" href="/docs/operations/plan/hardware">&lsaquo; Server Hardware</a>
</div>
<div class="pull-right">
<a class="btn btn-primary" href="/docs/operations/plan/network">Network &rsaquo;</a>
</div>
<div class="clearfix"></div>

&nbsp;
