---
title: SSD Setup
description: Learn how to set up and optimize Flash Devices (SSD and NVMe) for maximum performance with the Aerospike database.
styles:
  - /assets/styles/ui/steps.css

---
<style>
ol.steps {
  padding-top: 1em;
}
</style>

<div class="row">
  <section id="ssd setup">
    Aerospike’s flash-optimized architecture was designed with modern flash devices (SSDs) in mind. It is crucial to understand that installing and setting them up is somewhat different from rotational drives. You should check the list of approved flash devices and see if your model is there. You may also want to run the Aerospike Certification Tool (ACT) within your environment, even if your SSD is on the <a href="/docs/operations/plan/ssd/ssd_certification.html#approved_sata">approved list</a>.
    <p>
    When reading or writing records, Aerospike uses direct IO to the drive – Aerospike uses Linux’s direct (or raw device) interface in order to achieve high performance. There is no filesystem on the data drive, so there is no need to RAID or stripe the drive. (There does need to be a filesystem on any drive holding a [flash index](/docs/operations/configure/namespace/index/index.html#flash-index), however.)
    <p>
    There are many factors that will affect flash performance. Please note that these instructions are for tuning flash for use with Aerospike. These methods may differ from other uses.
    <p>
    <h3>Proper flash configuration</h3>
    The most important factors for any given flash device are (in very roughly descending order of importance):

    {{#steps}}

      {{#steps-step 1 "Do not use disks that are unevenly worn" markdown=true}}
Unlike rotational drives, flash devices are affected by rewriting over the same area again and again. While Aerospike wears through the drive evenly, if the drive was used for another purpose in the past, it may have worn areas where the performance will suffer dramatically. This is particularly true if the SSD is in a cloud environment. If you don't know how the drive was used, you should assume it was unevenly worn.
      {{/steps-step}}

      {{#steps-step 2 "Use a good model flash device" markdown=true}}
Flash device models vary greatly in performance. They even vary a lot depending on the use case. Proper selection of one is of great importance.
Please see the <a href="/docs/operations/plan/ssd/ssd_certification.html"> approved flash device list</a> for more information.
      {{/steps-step}}

      {{#steps-step 3 "Connect the drives properly" markdown=true}}
Even the best flash will be hampered by a poor controller. The best way to connect flash devices to the motherboard is without RAID. This can be through a SATA controller directly on the motherboard or through a controller that allows direct access to the drive. One useful thing to note is that Aerospike has generally found good results for disk controllers specifically designed for Apache Hadoop. For example, Aerospike has tested the HP P420 and found very good results. In addition, if your RAID controller uses the LSI 2208 chip, you may be able to take advantage of the the StorCLI (AKA MegaCLI) package. This has shown to improve performance dramatically. See the <a href="/docs/operations/plan/ssd/lsi_megacli.html">using StorCLI (AKA MegaCLI) page</a> for more info.
<p>
If your server requires the use of RAID, it is best to configure each flash device as its own RAID 0 array. You should configure the arrays with write-through mode and no-read ahead for the cache policy. The preferred method for installing your flash devices is with direct-connect or with a RAID controller in pass-through mode (i.e. not using RAID features). We do not recommend RAID with Aerospike Database.
<p>
{{/steps-step}}   

{{#steps-step 4 "Over-provision your flash devices" markdown=true}}
Flash over-provisioning sets aside space for the drive controller to do its work. You can think of it as reserved free space to move data around. When this space is too small, the controller must do much more work and thus will take more time to do its job. Some flash devices (such as the Intel S3700) come with enough free space and no changes are necessary. Others (such as the Intel S3500) will need additional over-provisioning to maintain peak performance under load. In the Aerospike approved list, we provide whether or not this was required. For those disks that require additional over-provisioning, we normally recommend 21%. So a 512 GB SSD will be about 400 GB after the over-provisioning.
<p>
There are 2 ways to over-provision flash devices.
<ul>
  <li>Using the Host Protected Area (HPA). This is a low-level way to protect an area for use by the controller. However, it may not be possible to set this on flash devices behind a RAID controller. You may either set the HPA from another machine and move it or use disk partitions (next).
  <li>Using disk partitions. In this case you will actually be creating a partition that is smaller than the full size of the disk and using the partition rather than the disk. For many flash devices (such as those from Intel and Samsung), this will have the same impact as setting the HPA. Not all SSDs will make use of unpartitioned space as if it were reserved for over-provisioned use. Check with your manufacturer if you are not sure.
</ul>

<p>Refer to the instructions on <a href="/docs/operations/plan/ssd/ssd_op.html">how to over-provision</a>.
{{/steps-step}}

{{#steps-step 5 "Partition your flash devices" markdown=true}}

<p>Some definitions:
<ul>
  <li>Partition - this is the standard definition of partition for all hard disks, whether they are rotational or SSD.
  <li>MBR - the Master Boot Record. This is the mechanism used for many years to contain information on how the disk is to be partitioned. This has a limit of 2 TB. There is a limit of 4 physical partitions and 15 logical partitions when using MBR.
  <li>GPT - the GUID Partition Table. This is the replacement for MBR. It (GPT) can coexist with MBR. This gets past the biggest limits of MBR. It allows for 127 partitions and a maximum size of 2^64 sectors. With 512B sectors, this translates to 9.4 ZB.
  <li>Device - This can get confusing as it can refer to either an entire disk or a single partition on one. For the purposes here, device refers to a single device declaration within the OS layer (e.g. /dev/sdb or /dev/nvme0n1)
</ul>


<p>Independently of the over-provisioning related points mentioned earlier, the following points should be considered when deciding how many partitions to create on a Flash (SSD) device:
<ul>
<li>Aerospike supports devices over 2TiB by requiring you to partition the physical device into smaller partitions. It may also be beneficial to partition 2TiB or smaller devices in order to increase parallelism as well as the number of device specific threads. Therefore, even more important than the partition size, it is the number of partitions that will drive the performance. Aerospike's best practices suggest benchmarking different configurations and keeping the number of partitions usually at most equal to the number of CPU cores. For example, on a 16 core system, it would be suggested to have 16 partitions of 800GiB each rather than 32 of 400GiB. But on a 32 core system, it may be beneficial to go up to 32 partitions of 400GiB each, potentially even higher, depending on the workload. Having too many partitions on the same physical device could also adversely impact performance. It is always recommended to fully benchmark the production workload at peak level on a small system in order to validate the configuration.
<li>Partitions on an SSD device do not have to be the same size. However, Aerospike does not treat them differently within a given namespace. So if you have a 100 GB partition and a 200 GB partition in the same namespace, the system will start running into performance issues when the 100 GB one reaches < 5% available mark and will get into stop_writes, as soon as the smallest partition hits the threshold. Therefore, it is necessary to keep devices in the same namespace the same size. This limitation does not apply between different namespaces. However, the total speed of the device will be the same across the namespaces. So if you have 2 different partitions with 2 different namespaces, one not having a lot of traffic, but the other one having tremendous traffic, the fast one may impact the load on the slower one.
<li>Having multiple partitions on an SSD device improves performance on cold start for large capacity devices. Cold starting a namespace requires the whole underlying storage device to be scanned, hence having more partitions allows for parallelizing this effort. For a given device capacity, results may differ between NVMe and SATA devices.
<li>While the raw read/write performance should not be impacted by the number of partitions, there is one thing that would affect it. Aerospike must do defragmentation at the application layer. This is currently single threaded per partition and the requirements for this are based on how fast you are writing to the system. Having more partitions increases the number of threads for the defragmentation process (one thread per partition).
</ul>

{{/steps-step}}

{{#steps-step 6 "Initialize the drives" markdown=true}}
If you are ready to use the flash devices, you will need to <a href="/docs/operations/plan/ssd/ssd_init.html">initialize them</a> before using them with Aerospike DB.
{{/steps-step}}


{{/steps}}

{{#warn markdown=true}}**Sharing of devices across namespaces is not allowed.**
The Aerospike process aborts by logging the following warnings if the same device is shared across namespaces:

```
Jun 04 2018 15:31:31 GMT: WARNING (drv_ssd): (drv_ssd.c:2466) read header: device /dev/ram0 previous namespace test now bar, check config or erase device
Jun 04 2018 15:31:31 GMT: FAILED ASSERTION (drv_ssd): (drv_ssd.c:3202) unable to read disk header /dev/ram0
Jun 04 2018 15:31:31 GMT: WARNING (as): (signal.c:196) SIGUSR1 received, aborting Aerospike Enterprise Edition build 4.1.0.1 os ubuntu14.04
```
{{/warn}}

<p>**Example of how to partition a device**<br>

Zero out the disk, if it has been previously used for another purpose (all data will be discarded from that disk):

```asciidoc
# ssd, if blkdiscard is available:
$ blkdiscard /dev/sdb
# OR all other disks, or if blkdiscard is not available:
$ dd if=/dev/zero of=/dev/sdb bs=131072

Open fdisk against /dev/sdb

$ sudo fdisk /dev/sdb
Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Create a new DOS/GPT partition table, if one doesn't exist yet (note - existing data, if any, will be overwritten on /dev/sdb). GPT (preferred):

Command (m for help): g
Created a new GPT disklabel (GUID: 7A193027-381B-4712-8D76-117908120FF7).
DOS (in case GPT is not available):

Command (m for help): o
Created a new DOS disklabel with disk identifier 0xfe6c3bc5.
Create 2 new partitions, of example size 500GiB each

Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-2097118, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-2097118, default 2097118): +500G

Created a new partition 1 of type 'Linux filesystem' and of size 500 GiB.

Command (m for help): n
Partition number (2-128, default 2): 
First sector (1026048-2097118, default 1026048):

Last sector, +sectors or +size{K,M,G,T,P} (2048-2097118, default 2097118): +500G

Created a new partition 2 of type 'Linux filesystem' and of size 500 GiB.
Write changes to disk:

Command (m for help): w
The partition table has been altered.
Synching disks.
```

{{#note}}If you receive an error with writing changes, don't see "syncing disks", or don't see /dev/sdb{1,2} devices following that, you will need to reboot the machine as the kernel partition refresh failed.
{{/note}}



  </section>

  <section id="next">
    <h3>Next steps</h3>
    <ul>
    	<li> <a href="/docs/amc">Install the Aerospike Management Console (AMC)</a>
      <li> <a href="/develop">Develop your app</a>
    </ul>
  </section>

</div>
