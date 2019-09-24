---
title: Startup Problems
---
 
The asd process (daemon) is the main Aerospike process that started, using the start command `/etc/init.d/aerospike start`.

On CentOS 6, this script can be used with the `/sbin/service` command.

### The asd daemon will not start
If you try to start the asd and get the following error:

```
touch: cannot touch `/var/lock/subsys/aerospike’: Permission denied
```

Confirm you are starting the process as the correct user with the appropriate permissions, using sudo when needed.

{{#info}}You must be root to start the daemon.{{/info}}
 
### Problem with network interface
If the server won't start due to inability to get the physical address,
and you see the following message in the log file:

```
Jun 22 2014 02:34:10 GMT: WARNING (cf:misc): (id.c::249) Tried eth,bond,wlan and list of all available interfaces on device.Failed to retrieve physical address with errno 19 No such device
Jun 22 2014 02:34:10 GMT: CRITICAL (config): (cfg.c:3363) could not get unique id and/or ip address
Jun 22 2014 02:34:10 GMT: WARNING (as): (signal.c::120) SIGINT received, shutting down
Jun 22 2014 02:34:10 GMT: WARNING (as): (signal.c::123) startup was not complete, exiting immediately
```

Check the name of your network interface by running the following command: 

```bash
ifconfig -a
```

You would need to specify the [`node-id-interface`](/docs/reference/configuration#node-id-interface) in the configuration.

{{#info}}
For versions prior to 3.10, you would need to specify the [`network-interface-name`](/docs/reference/configuration#network-interface-name) which was located in the `network.service` configuration context.
{{/info}}

E.g: the interface name in the example is p2p1:

```
    service {
        ...
    		  node-id-interface p2p1
        ...
    }
    ...
```
For more information on the configuration, see [Configuration Reference](/docs/reference/configuration).

### Not enough file descriptors error in log
Look at the Aerospike log and check if you see the following message:

```
Aug 24 2012 16:43:10 GMT: INFO (as): (base/as.c:172) File descriptor limit is : 1024 and proto-fd-max is : 2048
Aug 24 2012 16:43:10 GMT: CRITICAL GLOBAL (as): (base/as.c:174) Not enough file descriptors, Starting with 1024 and needs 2048
critical error: backtrace: frame 0 /usr/bin/asd() [0x460cef]
critical error: backtrace: frame 1 /usr/bin/asd() [0x404b59]
critical error: backtrace: frame 2 /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xed) [0x7f620b53030d] critical error: backtrace: frame 3 /usr/bin/asd() [0x403e79]
```

To resolve this, change the proto-fd-max setting in the [service section of your config file](/docs/reference/configuration) to be 100000, or increase the open file descriptor limit on your dev environment. 

### Stuck in Defrag loop at startup
When Aerospike starts up it requires some percentage of storage available to join the cluster. We recommend keeping the SSD at 50% utilization to allow efficient defragmentation.  The minimum percentage of storage that is required at startup is depends on the configuration, which is typically set to 10%: 

```
defrag-startup-minimum 10
```

If a node does not start up for a long time and the log file at /var/log/aerospike/aerospike.log shows something like:

```
Aug 22 2012 21:16:38 GMT: INFO (as): (base/as.c:265) waiting for defrag: namespace devices percent 0 waiting for 10
Apr 24 2013 17:19:59 GMT: INFO (drv_ssd): (storage/drv_ssd.c:1544) read_bin: could not read first
Apr 24 2013 17:20:00 GMT: WARNING (drv_ssd): (storage/drv_ssd.c:1390) **** ssd_read: record f8de4ae1d039c87 has no block associated, fail
Apr 24 2013 17:20:00 GMT: WARNING (drv_ssd): (storage/drv_ssd.c:1390) **** ssd_read: record f8de4ae1d039c87 has no block associated, fail
```

When the server starts, it tries to defrag and it won't start until it has defragged enough space to get the space available to match the startup minimum.  If the SSD is at too high a use percentage, the node may not be able to get enough free contiguous space to startup.

To get out of the defrag loop:

- **For an in-memory database with persistence**, in general, the filesize should be 8x the amount of memory (e.g., if the memory for data is 20 GB, then the filesize should be 160 GB).  Increase the filesize to the 8x limit or higher.  If your database has very high traffic, you may require higher than 8x.
- **To resolve this for nodes with SSDs**, lower the high-water-memory-pct in the [Aerospike configuration file](/docs/operations/configure/namespace/retention). Lower the  high-water-memory-pct, so it starts evicting objects to create some free space. If the node still does not startup/does not evict, lower the high water percentage more so the node will evict more records and the server will be able to get enough free space to startup.

Note that *eviction* is typically a back-stopping strategy – data should *expire* before you reach the high water marks that trigger eviction.  This problem is a symptom is a larger problem that you need to address:
- If your storage is insufficient, you need to re-evaluate your storage/capacity strategy, as described in [Managing Storage Capacity](/docs/operations/manage/storage).
- Contact Aerospike for the capacity planning spreadsheet to help you re-configure your cluster capacity.

### With a non-standard network device, the server won't start

On some distributions and with some unusual network devices, the server won't start and the log shows the following error:

```
Aug 22 2012 06:34:18 GMT: WARNING (cf:misc): (id.c:163) can’t get physical address, tried eth, bond, wlan. fatal: 19 No such device
```

This means the network interface name on the node is something other than “eth”, “bond”, or “wlan”.

Add the network-interface-name parameter to your configuration file.  In the example below, the device is named vlan708:

```
network {
           service {
                   address 10.0.2.131
                   port 3000
                   network-interface-name vlan708
                   } 
           }
```

If you have multiple Network Devices you must specify which one to use.

If you have multiple network devices, Aerospike will choose one of them (e.g., if you have eth0 and eth1, then Aerospike will choose one of them).  A common situation is a node that has two Ethernet ports, one for the Internet and one for internal traffic.  In this case, Aerospike needs to access the Internet port, but it may choose the wrong one, resulting in a node that does not see traffic correctly.   To specify a specific network device, add the access-address parameter to your configuration file.

### Warning Messages on starting Aerospike

If you start Aerospike and get the following warnings, note that they are related to memory management in the Linux kernel.

1.**Warning about SHMMAX:**

```
sudo service aerospike start
kernel.shmmax too low, setting to 1GB
kernel.shmmax = 1073741824
Starting aerospike:     [OK]
```

SHMMAX is the maximum size of a single shared memory segment. If you see the above output when you start a server node, it just means that the system was configured with a shared memory maximum block size that's less than the 1GB required by Aerospike.  The start script dynamically raises the limit to 1GB.  

Unless the machine is rebooted, you should only see this happen on the first start.

2.**Warning about SHMALL:**

```
sudo service aerospike start
kernel.shmall too low, setting to 4G pages
kernel.shmall = 4294967296
Starting aerospike:     [OK]
```

SHMALL is the sum of all shared memory segments on the whole system. If you see the above output when you start a server node, it means the start script dynamically raised the maximum number of shared memory pages.  Again, unless the machine is rebooted, you should only see this happen on the first start.  It's possible to see both limits raised during a single (first) start.

### Other problems
For other problems, be sure to check for consistent settings between all nodes in the cluster, including service and network settings.  Make sure that the namespaces are configured the same on each node.

Make sure that no firewall is interfering with communication between nodes (ports 3000 through 3004).
