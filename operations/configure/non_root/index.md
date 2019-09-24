---
title: Running As Non-Root
description: Learn how to set-up Aerospike to run as a non-root user.
---
 
This document describes how to set-up Aerospike to run as a non-root user. In this document we will refer to &lt;AerospikeUser&gt; and &lt;AerospikeGroup&gt; as the user and group. You may replace these names to the particular user/group you would like to use. The deb/rpm packages create a "aerospike" user and group which you may use. This document covers the following use cases:

- Configure a freshly installed Aerospike node.
- Change an existing Aerospike node.

Please note that the server needs to be started by a user with sudo permissions/root user. The running process is owned by the user defined in the aerospike config.

### Configure a Freshly Installed Aerospike Node
#### Install Aerospike
Install the Aerospike-Server package if you haven't done so already.  Detailed instruction can be found in the [install documentation](/docs/operations/install).

```
$ rpm -Uvh aerospike-server-<version>.x86_64.rpm
```
This installs the Aerospike files and directories and creates the user/group aerospike/aerospike. The default user/group for these files and directories is aerospike/aerospike. You may use this user/group or an alternative.

**BEFORE** starting up the Aerospike node, please make sure the following steps are followed:

#### Configure User and Group to Use
Edit Aerospike configuration file at /etc/aerospike/aerospike.conf.  In the service stanza, change user to &lt;AerospikeUser&gt; and group to &lt;AerospikeGroup&gt;.

```
service {
    user <AerospikeUser>
    group <AerospikeGroup>
}
```

{{#warn}}If upgrading to Aerospike Server 4.5.3.2 or newer, the modifications to /etc/systemd/system/aerospike.service.d/user.conf step below should not be configured.{{/warn}}

For **RHEL 7, Ubuntu 16.04 or 18.04, Debian 8+** (distributions based on systemd) running versions prior to 4.5.3.2, the following additional steps are required:
```
cat > /etc/systemd/system/aerospike.service.d/user.conf <<EOF
[Service]
User=AerospikeUser
Group=AerospikeGroup
EOF
```
Refer to this Knowledge-Base: https://discuss.aerospike.com/t/changing-usergroup-of-asd-process-under-systemd/4734 article for further details.


#### Configure Pid File
{{#note}}If running systemd there is no need to configure the pidfile.{{/note}}

In the service stanza of /etc/aerospike/aerospike.conf, ensure the pidfile specified is a file that &lt;AerospikeUser&gt; can write to, and &lt;AerospikeUser&gt; has creation permission on the enclosing directory. By default, this is set to /var/run/aerospike/asd.pid . 

```
service {
    ...
    pidfile /var/run/aerospike/asd.pid
    ...
}
```

There are multiple ways to ensure that "/var/run/aerospike" has file creation and writing permission. The easiest way is to change directory owner to &lt;AerospikeUser&gt;:&lt;AerospikeGroup&gt;:

```
$ chown -R <AerospikeUser>:<AerospikeGroup> /var/run/aerospike
```

Other possibilities include:
Changing the directory permission for group to be create and write for group, and add &lt;AerospikeUser&gt; to the group.
Changing the directory permission to allow all users to create and write in the directory.

{{#warn}}**Ubuntu 12.04** onwards mounts /var/run as tmpfs. This causes the /var/run/aerospike folder to be deleted on reboot. The folder is recreated by init script and the permissions are fixed for aerospike user. If you run your server as some other user, you should update the ASD_USER in init script (/etc/init.d/aerospike) to reflect your new user.{{/warn}}

#### Configure Logging
In the logging stanza of /etc/aerospike/aerospike.conf, ensure the file specified is a file that &lt;AerospikeUser&gt; can write to, and &lt;AerospikeUser&gt; has creation permission on the enclosing directory. By default this is set to /var/log/aerospike/aerospike.log .

```
logging {
    # Log file must be an absolute path.
    file /var/log/aerospike/aerospike.log {
        context any info
    }
}
```

#### Configure File Resources used by Namespace
If your namespace data is set up to be persisted to a file, please ensure that &lt;AerospikeUser&gt; can write to the file, and &lt;AerospikeUser&gt; has creation permission on the enclosing directory:

```
namespace bar {
    ...
    storage-engine device {
        file /opt/aerospike/data/bar.data
        ...
    }
}
```

#### Configure SSD Resources used by Namespace
If you are using any SSDs as raw devices, you will need to either add &lt;AerospikeUser&gt; to the disk group:

```
sudo usermod -a -G disk <AerospikeUser>
```

or add a udev rule to the Aerospike user giving it ownership of the devices used.

Open /etc/udev/rules.d/99-aerospike.rules and add a rule similar to the following:

```
KERNEL=="sd[bc]", OWNER="<AerospikeUser>"
```

This particular rule sets the devices /dev/sdb and /dev/sdc to be owned by &lt;AerospikeUser&gt;. Save this file and then reload and trigger the udev rules.

```
$ udevadm control --reload-rules
$ udevadm trigger
```

SSDs that are being used as filesystems (for example, for a [flash index](/docs/operations/configure/namespace/index/index.html#flash-index)) will need to have the directory ownership or permissions changed in the usual way.

#### Configure SSD Scheduler
Aerospike configuration allows for automatic SSD device scheduler setting. If running as non-root user, this functionality will no longer work. You will need to explicitly set the devices' scheduler mechanism to be "noop". For more detail, please see [SSD Initialization](/docs/operations/plan/ssd/ssd_init.html).

#### Configure Additional Directories
There are some additional directories created by Aerospike Installation. These directories' permissions also require changes to allow file creation and writing within the directories:

```
$ chown -R <AerospikeUser>:<AerospikeGroup> /opt/aerospike/smd # Used for persisting System Meta Data
$ chown -R <AerospikeUser>:<AerospikeGroup> /opt/aerospike/usr # Used for persisting User-Defined-Functions
```

#### Configure XDR
If you are using XDR, please also make sure the relevant files and parent directories have desired create and write permission:

Below lists the default XDR configuration:

```
# XDR
xdr {
    enable-xdr                   true
    namedpipe-path               /tmp/xdr_pipe
    digestlog-path               /etc/aerospike/digestlog 10000000000
    errorlog-path                /var/log/aerospike/asxdr.log
    xdr-pidfile                  /var/run/aerospike/asxdr.pid 
    ...
}
```

#### Checking for Success
Once the changes are made, you are ready to start the node.

{{#info}}{{#markdown}}**Starting the Server**

Aerospike server can be started only by root user or by a user with sudo permissions. The running process however is owned by the user defined in the config.

To start the server, use "sudo /etc/init.d/aerospike start"{{/markdown}}{{/info}}

 
Look at the log file to ensure that resources have all been started successfully.

### Change an Existing Installation
If you are running a node with a user and group that have root privileges, and wish to switch to a user and group without root privileges, you will need to stop the asd process, change the user and group in the Aerospike configuration file, change ownership and/or permissions for all relevant Aerospike resources so they can be accessed by the new user and group, then restart the process.

For the following directories, permissions need to be changed to allow file creation and writing within the directories.  The simplest way to do this is to change ownership of the directories to NewUser:NewGroup.

1. Change permissions for /opt/aerospike/smd/ (which contains system metadata).
2. Change permissions for /opt/aerospike/usr/ and all subdirectories (which contains user defined functions).
3. Change permissions for /var/log/aerospike/ and /var/run/aerospike/ (which contain the log file and pid file respectively).
4. If you are using file storage, change the permissions for /opt/aerospike/data/.

{{#warn}}If upgrading to Aerospike Server 4.5.3.2 or newer, the modifications to /etc/systemd/system/aerospike.service.d/user.conf step below should not be configured.{{/warn}}

For **RHEL 7, Ubuntu 16.04 or 18.04, Debian 8+** (distributions based on systemd) running versions prior to 4.5.3.2, the following additional steps are required:
```
cat > /etc/systemd/system/aerospike.service.d/user.conf <<EOF
[Service]
User=AerospikeUser
Group=AerospikeGroup
EOF
```
Refer to this Knowledge-Base: https://discuss.aerospike.com/t/changing-usergroup-of-asd-process-under-systemd/4734 article for further details.

For the following files and resources, to get the node running as if it had originally been run with the new user and group, ownership must be changed.

1. Change ownership & permissions for all files in the directories listed above.
2. Shared memory.  Remove the existing shared memory blocks owned by Aerospike.  For deployments that [fast restart](/docs/operations/manage/aerospike/fast_start), this will force a cold restart, and the new shared memory blocks will be created by the new user and group.  If you wish to avoid a cold restart, it is possible to change ownership of the shared memory using a special program and script.  For more information, refer to the [Fast Restart](/docs/operations/manage/aerospike/fast_restart.html) page.
3. If you are using raw device storage, see section on changing ownership & permissions for devices.

Finally, if you are using raw devices and running without root privileges, you will not be able to use the configuration file option "scheduler-mode".  Instead you must make sure to set the device scheduler mode. For more information, please see [SSD Initialization](/docs/operations/plan/ssd/ssd_init.html).

If any of these steps are omitted, the asd process will self-terminate on restart, with a log message suggesting which resource requires permission changes.
