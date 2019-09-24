---
title: SSD Over-Provisioning
description: Follow these instructions to over-provision Flash (SSD)s. 
---

## Over-provision using HPA

{{#note}}{{#markdown}}
Use the instructions here if you can directly access the flash device. Most RAID controllers prevent direct control. For some drives, turning off HPA may be difficult, if not impossible. Use partitions to over-provision instead.
{{/markdown}}{{/note}}

### Install a recent version of hdparm (version 9.37+)

Some operating systems have a version of hdparm already installed.  We require version 9.37 or higher.  If you do not have a good version of hdparm, you can download and install as follows:
 
```bash
wget "downloads.sourceforge.net/project/hdparm/hdparm/hdparm-9.43.tar.gz"
tar -zxvf hdparm-9.43.tar.gz
cd hdparm-9.43
make
```

### Validate that the drive is not frozen

Check to see if the drive is frozen with this command:

```bash
sudo ./hdparm -I /dev/<deviceID>
```

If it’s frozen then unplug the drive for a few seconds and replug them or if you do not have access to physical machine then just suspending the machine for few seconds will remove it out of frozen state.

The command to bring any drive out of frozen state is:

```bash
rtcwake -m mem -s 180
```

### How much to over-provision
Determine the value to use for over-provisioning by issuing the `hdparm -N` command.  This command returns a fraction max sectors that indicates the maximum number of sectors that are available to the operating system – i.e., the numerator is the available space and denominator is the size of drive. For example:

```bash
$ sudo hdparm -N /dev/sdb
/dev/sdb:
 max sectors   = 468862128/468862128, HPA is disabled
```

Calculate the over-provisioning value by multiplying the denominator by 79%.  For example, in this case, we would calculate: 468862128 x 0.79 = 370401081.

Set the over-provisioning:

```bash
sudo ./hdparm -NpXXXXXXXX --yes-i-know-what-i-am-doing /dev/<deviceID>
```

`XXXXXXXX` is the number of sectors to over-provision, as calculated above.

Once you have over-provisioned all of your SSDs, restart the machine.

{{#note}}It is very important to restart the machine after over-provisioning using hdparm. Not doing so can have unpredictable outcomes.{{/note}}

After the machine is restarted, confirm that the SSD(s) are correctly over-provisioned by using the following command for each drive:

```bash
sudo ./hdparm -N /dev/<deviceID>
```

returns "`HPA is enabled`" with the  correct fraction. For example, you might see:

```bash
$ sudo ./hdparm -N /dev/sdb
/dev/sdb:
 max sectors   = 370401081/468862128, HPA is enabled
```

## Over-provision using Partitions

This page describes how to do basic drive installation when you are using a RAID controller and RAID features, or if you simply prefer using partitions.

{{#note}}{{#markdown}}
If you are using Amazon EC2 instances with Flash/SSDs (such as the i2 instances), setup is done for you by Amazon, so you can skip this installation step and proceed to <a href="/docs/operations/plan/ssd/ssd_init.html">flash device initialization.</a>

If you are using direct connect or your RAID controller is set in pass-through mode (JBOD), use the setup instructions for "Over-provision with Host Protected Area (HPA)" by selecting it above.

Some manufacturers (such as Micron/Crucial) do not make use of unpartitioned space for over-provisioning. So test to see if your drives will work.
{{/markdown}}{{/note}}

#### Set up RAID controllers  

Be sure that the following configuration steps are done: 

- Set up the flash devices connected to the RAID controller as separated devices configured with RAID 0 with 128KB strips (use <a href="/docs/operations/plan/ssd/lsi_megacli.html">LSI's StorCLI (AKA MegaCLI)</a>, if possible)
- Set the Read Policy as No Read Ahead
- Set the Write Policy as Write Through
- Enable NCQ/AHCI if that option is available


#### Delete any existing partitions using fdisk

Review the partitions for the SSD device(s), typically sdb, sdc, etc. with this command: 

```bash
sudo /sbin/fdisk /dev/<deviceID>
```

1. Look at the partition table by entering: `p`
2. Delete the partition (if one exists) by entering: `d`
3. Commit the changes and exit: `w`

### Over-Provision with fdisk

Over-provisioning reserves a portion of the flash device for use by the disk controller, to allow for high speed operation. Aerospike recommends that a total of 29% of the disk be "over-provisioned" (set aside) to maintain performance.  Consumer-rated drives typically set aside 8% and need to be overprovisioned by an additional 21%.  Enterprise-grade flash devices typically set aside enough, so you do not need to do any additional over-provisioning.  Check with your flash device manufacturer to determine whether your drive is over-provisioned and by how much. If this is not necessary, you may skip the over-provioning step and go straight to <a href="/docs/operations/plan/ssd/ssd_init.html">initializing the drives.</a>

If your flash device requires over-provisioning, you must create a disk partition for the disk space that is to be used for data storage.  The typical case is to create a partition that is 79% of the rated size, to ensure that 21% of the disk is reserved for the disk controller.
To create a disk partition use this command:

```bash
sudo /sbin/fdisk /dev/<deviceID>
```

1. To create a new partition enter `n`
2. To make it the primary partition enter `p`
3. For the partition number enter `1`
4. Specify the first cylinder enter `1`
5. Specify the last cylinder — multiply the proposed (default) value by 0.79
6. Verify the partition table by entering `p`
7. Commit the changes by entering `w`

The fdisk command sequence will look something like this:

```bash
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-19140, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-19140, default 19140): 15121
 
Command (m for help): p
Disk /dev/sdb: 157.4 GB, 157437394944 bytes
255 heads, 63 sectors/track, 19140 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xeff8f3ae
   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1       15121   121459401   83  Linux
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
```

Do the above steps for all drives.

Reboot if necessary for partitions to be recognized. `fdisk` will indicate whether or not a reboot will be necessary after committing each partition change.

Using partitions for over-provisioning does not require restarting the server.

