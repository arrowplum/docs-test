---
title: asmonitor - Common Tasks
description: Learn additional details of the more common tasks for which asmonitor is used, such as checking cluster health and changing configuration parameters.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: Aerospike Monitor
    url: /docs/v3/Aerospike Monitor.html
---
{{#note}}
Deprecated from Aerospike Tools Versions >= 3.9.0. Use asadm instead.
{{/note}}

### Checking Cluster Health

The most common task in asmonitor is to look at the state of the cluster with the info command.  That tells you:

1. A list of all of the nodes in the cluster – shows whether they are in agreement as to the cluster state (i.e. Cluster Visibility shows true for all nodes)
2. Namespace information – shows per namespace the statistics:  how much memory is being used, how many records are stored in the namespace

The info command is described in detail here:  asmonitor – Displaying Tables.

### Viewing/Changing Configuration Parameters

To change any dynamic settings for the entire cluster use the asinfo command from the asmonitor console:
```bash
asinfo -v [get/set configuration commands] [-h <comma separated hostip:port list>] [-p default port. 3000 assumed if missing]
```

This command operates on ALL nodes, unless you use the -h parameter or the-p port values to specify only certain nodes.  The command strings are the standard asinfo commands.

### Running a Single Command

If you want to run a single command you can use the -e option.  For example:
```
asmonitor -e "info Node"
```
displays Node statistics without starting the console.

{{#note}}The default [tables](/docs/tools/asmonitor/tables.html) are Node, Namespace and XDR. These can be run individually like  `asmonitor -e "info XDR"` {{/note}}


### The Console Interface

asmonitor is a console which runs various commands.

asmonitor allows you to edit/repeat, like the bash shell.  You can enter commands more quickly using bash-like key sequences:

- Control-P or up arrow key brings back previous command(s), 
- Control-N or down arrow key moves forward through the command list,
- Control-F or right arrow key moves the cursor to the right non-destructively, 
- Control-B or left arrow key moves the cursor to the left non-destructively,
- Control-R searches history,
- `<tab>` autocompletes commands wherever available,
- ? shows help,
- ! sends shell commands to the operating system.

### Example asmonitor Session

Here is an example of running `asmonitor` on a one node cluster:

```
$ asmonitor -h 127.0.0.1:3000
 
Enter help for help
 
1 host names are discovered in the cluster
Citrusleaf Interactive Shell, version 2.1.3
 
Monitor>
Monitor> info
===NODES===
2012-12-21 13:20:10.454209
ip:port                 Build   Cluster      Cluster   Free   Free   Migrates              Node   Replicated    Sys
                            .      Size   Visibility   Disk    Mem          .                id      Objects   Free
                            .         .            .    pct    pct          .                 .            .    Mem
192.168.105.168:3000    2.1.3         1         true      0     99      (0,0)   BB93E0303CA0568      0.100 M     98
Number of nodes displayed: 1
 
 ===NAMESPACE===
Total(unique) objects in cluster for usermap : 0.100 M
 
ip/namespace               Avail   Evicted   Objects     Repl     Stop   Used   Used     Used   Used    hwm   hwm
                             Pct   Objects         .   Factor   Writes   Disk   Disk      Mem    Mem   Disk   Mem
                               .         .         .        .        .      .      %        .      %      .     .
192.168.105.168/usermap       99         0   0.100 M        1    false    n/a    n/a   6.10 M      1     95    95
Number of rows displayed: 1
 
 ===XDR===
Nodes                   Build      Bytes   Free     Lag           Req    Req.        Req.   Uptime
                            .    Shipped   dlog    secs   Outstanding   Relog     Shipped        .
192.168.105.168:3004    2.1.3   256.73 M    18%       0          2300   1,416   4,016,552   77,977
Node of XDR responding: 1
```

### Latency Tracking

From the asmonitor console, to see latency statistics for the entire cluster enter the command:

```
latency
```

If latency tracking is not enabled on servers then use the command:

```
latency -c
```

to set latency configuration on the servers.

By default, the latency command will show all the histograms, i.e reads,writes, proxy and writes_reply. If you only want to see some histograms for the entire cluster, use the -k option to specify the histogram.  For example, the command

```
latency -k reads
```

displays reads for the entire cluster. There are many options you can use to customize your results.  Use the latency -u command to see usage:  

```
latency
 [-v <histogram name filter>]
 [-h <comma separated hostip:port list>]
 [-p default port. 3000 assumed if missing]
 [-k histogram key (reads, writes_master, writes, writes_reply, proxy)]
 [-d duraction_sec]
 [-v value spec for filtering with ? and * wildcard expressions]
 [-b number of seconds (before now) to look back to]
 [-d duration, the number of seconds from start to search]
 [-s slice_sec, i.e. interval in sec to analyze]
 [-t flag (want throughput). If present, display the throughtputs data instead]
 [-m flag (by machine). If present, display the output group by machine names also]
 [-c flag (config). If present, run the histogram configuration step]
Get the latency histogram data
```

e.g. If you would like to just see default latency for all nodes

```
Monitor> latency
    ====writes_master====
                                      timespan   ops/sec   >1ms   >8ms   >64ms
192.168.120.109:3000    00:10:01-GMT->00:10:11    1833.6   1.29   0.09    0.00
192.168.120.110:3000    00:10:01-GMT->00:10:11    1984.4   1.11   0.06    0.00
192.168.120.111:3000    00:10:06-GMT->00:10:16    2040.9   1.15   0.08    0.00
  
    ====reads====
                                      timespan   ops/sec   >1ms   >8ms   >64ms
192.168.120.109:3000    00:10:01-GMT->00:10:11    4432.1   0.85   0.02    0.00
192.168.120.110:3000    00:10:01-GMT->00:10:11    4546.7   0.83   0.06    0.00
192.168.120.111:3000    00:10:06-GMT->00:10:16    4629.1   0.89   0.04    0.00

  
    ====proxy====
                                      timespan   ops/sec   >1ms   >8ms   >64ms
192.168.120.109:3000    00:10:01-GMT->00:10:11       0.0   0.00   0.00    0.00
192.168.120.110:3000    00:10:01-GMT->00:10:11       0.0   0.00   0.00    0.00
192.168.120.111:3000    00:10:06-GMT->00:10:16       0.0   0.00   0.00    0.00

  
    ====writes_reply====
                                      timespan   ops/sec   >1ms   >8ms   >64ms
192.168.120.109:3000    00:10:01-GMT->00:10:11    1833.7   1.29   0.09    0.00
192.168.120.110:3000    00:10:01-GMT->00:10:11    1984.4   1.11   0.06    0.00
192.168.120.111:3000    00:10:06-GMT->00:10:16    2040.8   1.15   0.08    0.00

```

### Watching Statistics

In the `asmonitor` console, the `stat` command is the same as running `asinfo -v "statistics"` on all nodes. In addition, you can use wildcard expressions to get more specific statistics.  

For example:

```
Monitor> stat --help
stat [-v <Value to filter>]
     [-h <comma separated hostip:port list>]
     [-p default port. 3000 assumed if missing]
     [-v value spec for filtering with ? and * wildcard expressions]
Monitor> stat -v *expired*
    ====192.168.120.109:3000====
stat_expired_objects    19,313
    ====192.168.120.110:3000====
stat_expired_objects    21,000
    ====192.168.120.111:3000====
stat_expired_objects    24,218
```



You can use the `watch` command to run a asmonitor command every n seconds and display the results.The watch command loops continuously until you press Ctrl-C to terminate the repetition.


In the `asmonitor` console, the syntax is:

```bash
watch -n <seconds> <clmonitor command>
```

For example, if you want to watch the latency on the server or changes in some table, you would do:

```
Monitor> watch -n 10 latency -k reads
Every 10.0s: latency            2013-03-25 16:50:26.170088
  
    ====reads====
                                      timespan   ops/sec   >1ms   >8ms   >64ms
192.168.120.109:3000    23:50:08-GMT->23:50:18    4915.1   0.98   0.04    0.00
192.168.120.110:3000    23:50:08-GMT->23:50:18    4997.6   0.78   0.04    0.00
192.168.120.111:3000    23:50:13-GMT->23:50:23    5260.5   0.78   0.03    0.00
  
  
  
Every 10.0s: latency            2013-03-25 16:50:36.175412
  
    ====reads====
                                      timespan   ops/sec   >1ms   >8ms   >64ms
192.168.120.109:3000    23:50:18-GMT->23:50:28    4857.9   0.95   0.03    0.00
192.168.120.110:3000    23:50:18-GMT->23:50:28    4980.5   0.96   0.03    0.00
192.168.120.111:3000    23:50:23-GMT->23:50:33    5188.6   0.66   0.03    0.00
```
