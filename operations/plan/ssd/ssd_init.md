---
title: Flash (SSD) Initialization
description: Follow these easy instructions to initialize Flash (SSD and NVMe) with the Aerospike database. 
---

These instructions assume that you have already done the basic installation for the flash device (SSD).

### Erasing the drive

The drives must be erased or zeroed out before use.

Erasing drives is similar to formatting a conventional drive and may take an hour or more (depending on the drive) but the drives can be erased concurrently. To erase a flash device, use the command for each device:

```bash
sudo dd if=/dev/zero of=/dev/<deviceID> bs=1M&
```

Once the disk is completely dd'ed, dd will finish with the message `No space left on device` or an `out of space error`.  This shows that dd has successfully completed writing to the entire disk and there is no more space left for dd to continue.

To monitor the progress of dd, you can use:


```bash

bash$ while [ 1 ]
do kill -USR1 (pid_of_dd)
sleep 60
done

```


{{#warn}}Any pre-existing data on the disk will be zeroed out during this process. Be sure that you do not do this to the drive containing the operating system because it will erase the OS.{{/warn}}

{{#note}}Starting from version 3.3.5, you no longer need to dd the existing remaining disks when adding a new disk or replacing an existing disk with a new disk in an existing namespace.{{/note}}
	
{{#note}} Before version 3.3.5 , all the flash devices associated with a given namespace on a server must be initialized together.  Thus, if a new drive is being added to a namespace or you are replacing a failed drive, you must initialize ALL of the devices for that namespace with the dd process as well.{{/note}}
