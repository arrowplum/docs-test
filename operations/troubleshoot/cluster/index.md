---
title: Cluster Problems
description: 
---
 
### Migration
When recovering from a cluster integrity fault (because of a cluster size change), data migration occurs on the affected nodes. Migration threads take lower priority than the others, and can take a long time to complete depending on the environment and configuration.

Look at [Tuning Migration](/docs/operations/manage/migration/index.html#tuning-migrations) for more information.

### Cluster Size Change
If you suspect that a node may be down, you can check in the following ways:

1. Use the [asadm tool](/docs/tools/asadm) with the info command.
2. Check the server log and look for “CLUSTER SIZE = N”. This will tell you how many nodes that particular node thinks is in the cluster. Note that there are times when there may not be agreement. This can be true if a set of nodes has split from the main cluster. (This is sometimes referred to as a *split brain*.) Generally, it is easiest to restart the “lost” node(s) to get it/them to rejoin the cluster.


### Iptable Configuration
Customers may need to disable or re-configure IP table to ensure inter-node communications is not blocked.

To Turn OFF IPTables

IPTables will interfere with the normal operation of Aerospike. We recommend that you turn it off. 

You can run either:

```
sudo /usr/sbin/ntsysv 
 uncheck iptables so that service won't be automatically started
click Ok
```

or use chkconfig by executing

```
sudo /sbin/chkconfig iptables off
```

After you have disabled IPTables, you must turn it off by issuing the command:

```
sudo /etc/init.d/iptables stop
```


### The Cluster Appears Healthy but the Applications/Clients are Reporting Errors

If you stop seeing writes go to the cluster (as shown by the server logs) and the client is reporting errors/not fully happy but there are no errors in the server log,
you can check for these possible causes:

- Check service iptables status – to see if any port is blocked. Even in cases where the service had been up, there have been cases where the port was reset to be blocked.  
- Check clients/servers’ CPU load average. One of the causes may be a runaway process on the client or server. Please note that if one of the CPUs on the server is showing 100%, but the others are not, this may be normal behavior defrag or another process is running. This can be most commonly done using the Linux `top` command.
- Check to make sure heartbeat_received (foreign counter) is increasing/changing (and not stuck) using the  `asadm` tool and issue the `show statistics like heartbeat` command.  The node expects heartbeats from the other nodes and the foreign counter indicates how many heartbeats have being received from other nodes – the count should be constantly increasing.

**Note:** If you are using mesh, heartbeats received for self are always 0 (this is expected). In multicast, heartbeats received for self should be increasing/changing.
- Grep for the trans_in_progress keyword on the server log with this command: 

	```
	tail -200f /var/log/aerospike/aerospike.log |grep 'in-progress'
	```
	a sample output is shown below.
	- If the read-write counter (“rw-hash”) is not going down, it means requests are getting backed up. Check for growth pattern.
	- If proxy counter (“prox-hash”) is not zero, it means port is blocked for incoming requests. Check the fabric port 3001 (service iptables status)
	- If any of the queues are constantly increasing, it may hint of some potential issues on the node unable to keep up.

 ```	
Oct 03 2017 18:31:49 GMT: INFO (info): (ticker.c:267)    in-progress: tsvc-q 0 info-q 0 rw-hash 0 proxy-hash 0 tree-gc-q 0
Oct 03 2017 18:31:59 GMT: INFO (info): (ticker.c:267)    in-progress: tsvc-q 0 info-q 0 rw-hash 0 proxy-hash 0 tree-gc-q 0
Oct 03 2017 18:32:09 GMT: INFO (info): (ticker.c:267)    in-progress: tsvc-q 0 info-q 0 rw-hash 0 proxy-hash 0 tree-gc-q 0
Oct 03 2017 18:32:19 GMT: INFO (info): (ticker.c:267)    in-progress: tsvc-q 0 info-q 0 rw-hash 0 proxy-hash 0 tree-gc-q 0

 ```

### Validate Multicast

If you want to use multicast, you can validate that multicast is properly enabled on your network, or a particular multicast address is being correctly routed between particular machines.

Note that these tests will only test for the initial multicast connectivity. There are many reasons that your network equipment may initially allow multicast, but may halt it at a later time.

The best ways to ensure multicast runs smoothly are:

- Turn off storm control to the nodes. Often the connections on the cluster will look like a denial of service attack, so storm control may stop these connections. Check `dmesg` or `/var/log/messages` to identify if something specific is showing up as the cause of the error.

- In order to ensure group membership you should do ONE of the following:
	- Turn off IGMP snooping on the cluster vlan.
	- Turn on IGMP snooping AND turn on the querier (create a multicast router).


- Identify if there are any hops occuring in the packets sent between the nodes in the cluster. You can verify it by using the `traceroute` command. If you identify that there are multiple hops between nodes, you can attempt to increase the [mcast-ttl](/docs/reference/configuration/index.html#mcast-ttl) configuration to 2 or 3 on Aerospike configuration as a workaround.

- We recommend using a very simple multicast test tool called `open-mtools`, made available by [Google Code](https://code.google.com/archive/p/open-mtools/downloads) without restriction. Download and run the mtools (Note: mtools will not work properly in VM environments).

Install `mtools` on both the sending and receiving machines:

```
$ wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/open-mtools/mtools.1.0.zip
$ unzip mtools.1.0.zip
$ cd mtools
$ gcc mdump.c -o mdump; gcc msend.c -o msend
```
Run the following `mdump` on the receiving machines first:
```
$./mdump -Q2 -v 239.1.1.1 2345 <interface ip>
```

Run the `msend` command on a machine that will be broadcasting:
```
$ ./msend -m 100 -b 300 -n 600 -s 10 239.1.1.1 2345 <ttl> <interface IP>
Equiv cmd line: msend -b300 -m100 -n600 -p1000 -s10 -S65536 239.1.1.1 2345
Sending 600 bursts of 300 100-byte messages
Sending burst of 300 msgs
Sending burst of 300 msgs
Sending burst of 300 msgs
Sending burst of 300 msgs
Sending burst of 300 msgs
...
Pausing before sending 'stat
Sending stat
180000 messages sent (not including 'stat')
```

This will send 600 bursts of 300 100-byte messages on address 239.1.1.1 on port 2345. Aerospike is currently configured for a different address, but this test succeeding will show that multicast functionality is working.

See if a machine is receiving:
```
$ ./mdump -Q2 -s -v 239.1.1.1 2345
Equiv cmd line: mdump -p0 -Q2 -r4194304 -v 239.1.1.1 2345
WARNING: tried to set SO_RCVBUF to 4194304, only got 262142
echo sender equiv cmd: msend -b300 -m100 -n5 -p1000 -s10 -S65536 239.1.1.1 2345
180000 msgs sent, 180000 received (not including 'stat')
0.000000% loss
```

The normal test scenario would be to run msend from one machine and mdump on the rest of machines. You can change the parameter of `-b` to send more packets in one burst. `mdump` will print summary as follows which tells what is the %loss.

Verify multicast by testing sending and receiving on all nodes.

A reasonable validation scenario for four servers might be as follows:
- Run with node 01 as sender and nodes 02, 03, 04 as receivers and verified that the bytes are received fine.
- Repeating the test thrice each time using a different node as sender and the rest of the nodes as receivers and verified success.


These tests are recommended when a new cluster is setup or when a brand new machine is added or when a server moves within the data center or when network configuration changes.
