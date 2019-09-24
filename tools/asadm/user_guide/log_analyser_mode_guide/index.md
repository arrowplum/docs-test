---
title: Aerospike Admin Log-Analyzer Mode Commands
description: Introduction to Aerospike Admin Log-Analyzer mode commands.

---

### Help
The `help` command provides a brief description of all supported commands. You can
provide the name of a command to print help for that specific command.

In the example below, we request help for the `grep` command.

```asciidoc
Log-analyzer> help grep
Displays all lines from server logs matched with input strings.
  Options:
    -s <string>  - Space seprated Strings to search in log files, MANDATORY - NO DEFAULT
                   Format -s 'string1' 'string2'... 'stringn'
    -a           - Set 'AND'ing of search strings (provided with -s): Finding lines with all serach strings in it. 
                   Default is 'OR'ing: Finding lines with atleast one search string in it.
    -v <string>  - Non-matching strings (space separated).
    -i           - Perform case insensitive matching of search strings (-s) and non-matching strings (-v).
                   By default it is case sensitive.
    -u           - Set to find unique lines.
    -f <string>  - Log time from which to analyze.
                   May use the following formats:  'Sep 22 2011 22:40:14', -3600, or '-1:00:00'.
                   Default: head
    -d <string>  - Maximum time period to analyze.
                   May use the following formats: 3600 or 1:00:00.
    -n <string>  - Comma separated node numbers. You can get these numbers by list command. Ex. : -n '1,2,5'.
                   If not set then runs on all server logs in selected list.
    -p <int>     - Showing output in pages with p entries per page. default: 10.
```

### List
The `list` command displays aerospike server logs currently added and selected for actual processing. 

```asciidoc
Log-analyzer> list
Added Logs: 
1  : bb9635c2a270008     /var/log/aerospike/aerospike.log 


Selected Logs: 
     bb9635c2a270008     /var/log/aerospike/aerospike.log
```

### Add
The `add` command is used to add new log files from different nodes of the cluster to the log-analyzer. 
It adds the file to both all and selected list. Only one log file may be added per node. 
It requires space separated path of log or directory containing logs. For log file of server (version >=3.7.1), 
log-analyzer fetches the node id from log and set it as a display name, otherwise uses MD5_[MD5_hash_of_path].

Example Output:
```asciidoc
Log-analyzer> add /home/vagrant/shared/data/testlogs/ /var/log/aerospike/aerospike2.log
INFO: Added Log File /home/vagrant/shared/data/testlogs/files/1.log.
INFO: Added Log File /home/vagrant/shared/data/testlogs/files/2.log.
INFO: Added Log File /home/vagrant/shared/data/testlogs/files/3.log.
INFO: Added Log File /home/vagrant/shared/data/testlogs/files/4.log.
INFO: Added Log File /home/vagrant/shared/data/testlogs/files/5.log.
INFO: Added Log File /home/vagrant/shared/data/testlogs/files/6.log.
6 server logs added for server analysis.
INFO: Added Log File /var/log/aerospike/aerospike2.log.
1 server log added for server analysis.
Log-analyzer> 
Log-analyzer> 
Log-analyzer> list
Added Logs: 
1  : MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
2  : MD5_13e2b3a769b     /home/vagrant/shared/data/testlogs/files/5.log 
3  : MD5_41a561df934     /home/vagrant/shared/data/testlogs/files/3.log 
4  : MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
5  : MD5_ed54a4077e8     /home/vagrant/shared/data/testlogs/files/1.log 
6  : MD5_ef044710d3d     /home/vagrant/shared/data/testlogs/files/2.log 
7  : bb9635c2a270008     /var/log/aerospike/aerospike.log 
8  : c1d635c2a270008     /var/log/aerospike/aerospike1.log 
9  : c81635c2a270008     /var/log/aerospike/aerospike2.log 


Selected Logs: 
     MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
     MD5_13e2b3a769b     /home/vagrant/shared/data/testlogs/files/5.log 
     MD5_41a561df934     /home/vagrant/shared/data/testlogs/files/3.log 
     MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
     MD5_ed54a4077e8     /home/vagrant/shared/data/testlogs/files/1.log 
     MD5_ef044710d3d     /home/vagrant/shared/data/testlogs/files/2.log 
     bb9635c2a270008     /var/log/aerospike/aerospike.log 
     c1d635c2a270008     /var/log/aerospike/aerospike1.log 
     c81635c2a270008     /var/log/aerospike/aerospike2.log 

```

### Select
The `select` command provides a way to select server log from all list to selected list. 
It removes existing logs from selected list and adds newly selected logs.

Example Output:
```asciidoc
Log-analyzer> list
Added Logs: 
1  : MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
2  : MD5_13e2b3a769b     /home/vagrant/shared/data/testlogs/files/5.log 
3  : MD5_41a561df934     /home/vagrant/shared/data/testlogs/files/3.log 
4  : MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
5  : MD5_ed54a4077e8     /home/vagrant/shared/data/testlogs/files/1.log 
6  : MD5_ef044710d3d     /home/vagrant/shared/data/testlogs/files/2.log 
7  : bb9635c2a270008     /var/log/aerospike/aerospike.log 
8  : c1d635c2a270008     /var/log/aerospike/aerospike1.log 
9  : c81635c2a270008     /var/log/aerospike/aerospike2.log 


Selected Logs: 
     MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
     MD5_13e2b3a769b     /home/vagrant/shared/data/testlogs/files/5.log 
     MD5_41a561df934     /home/vagrant/shared/data/testlogs/files/3.log 
     MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
     MD5_ed54a4077e8     /home/vagrant/shared/data/testlogs/files/1.log 
     MD5_ef044710d3d     /home/vagrant/shared/data/testlogs/files/2.log 
     bb9635c2a270008     /var/log/aerospike/aerospike.log 
     c1d635c2a270008     /var/log/aerospike/aerospike1.log 
     c81635c2a270008     /var/log/aerospike/aerospike2.log 

Log-analyzer> 
Log-analyzer> 
Log-analyzer> select 1 4 9
Log-analyzer>     
Log-analyzer> list
Added Logs: 
1  : MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
2  : MD5_13e2b3a769b     /home/vagrant/shared/data/testlogs/files/5.log 
3  : MD5_41a561df934     /home/vagrant/shared/data/testlogs/files/3.log 
4  : MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
5  : MD5_ed54a4077e8     /home/vagrant/shared/data/testlogs/files/1.log 
6  : MD5_ef044710d3d     /home/vagrant/shared/data/testlogs/files/2.log 
7  : bb9635c2a270008     /var/log/aerospike/aerospike.log 
8  : c1d635c2a270008     /var/log/aerospike/aerospike1.log 
9  : c81635c2a270008     /var/log/aerospike/aerospike2.log 


Selected Logs: 
     MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
     MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
     c81635c2a270008     /var/log/aerospike/aerospike2.log 
```

### Remove
The `remove` command is used to remove logs from list of server logs. Use _all_ to remove all server logs.

Example Output:
```asciidoc
Log-analyzer> list
Added Logs: 
1  : MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
2  : MD5_13e2b3a769b     /home/vagrant/shared/data/testlogs/files/5.log 
3  : MD5_41a561df934     /home/vagrant/shared/data/testlogs/files/3.log 
4  : MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
5  : MD5_ed54a4077e8     /home/vagrant/shared/data/testlogs/files/1.log 
6  : MD5_ef044710d3d     /home/vagrant/shared/data/testlogs/files/2.log 
7  : bb9635c2a270008     /var/log/aerospike/aerospike.log 
8  : c1d635c2a270008     /var/log/aerospike/aerospike1.log 
9  : c81635c2a270008     /var/log/aerospike/aerospike2.log 


Selected Logs: 
     MD5_0b32a78b084     /home/vagrant/shared/data/testlogs/files/6.log 
     MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
     c81635c2a270008     /var/log/aerospike/aerospike2.log 

Log-analyzer> 
Log-analyzer> 
Log-analyzer> 
Log-analyzer> remove 1 3 7
INFO: Removed Log File /home/vagrant/shared/data/testlogs/files/6.log.
INFO: Removed Log File /home/vagrant/shared/data/testlogs/files/3.log.
INFO: Removed Log File /var/log/aerospike/aerospike.log.
Log-analyzer> 
Log-analyzer> 
Log-analyzer> list
Added Logs: 
1  : MD5_13e2b3a769b     /home/vagrant/shared/data/testlogs/files/5.log 
2  : MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
3  : MD5_ed54a4077e8     /home/vagrant/shared/data/testlogs/files/1.log 
4  : MD5_ef044710d3d     /home/vagrant/shared/data/testlogs/files/2.log 
5  : c1d635c2a270008     /var/log/aerospike/aerospike1.log 
6  : c81635c2a270008     /var/log/aerospike/aerospike2.log 


Selected Logs: 
     MD5_9dc865740e4     /home/vagrant/shared/data/testlogs/files/4.log 
     c81635c2a270008     /var/log/aerospike/aerospike2.log 
```


### Grep
The `grep` command is used to filter out lines containing a match to the input pattern, merge lines from different file on the basis of timestamp, and display in ascending order of timestamp. 
This command supports multiple input patterns, runtime server log selection, ignore-case, invert-match, uniq filter, and execution over specific time period. 

To avoid longer waiting time, we can use _-p_ option to set output page size. Its default value is 10, so after every 10 result entries it displays output.

```asciidoc
Log-analyzer> grep -s "reads" -n "5,6" -f "Jan 31 2017 07:00:00" -d 100
  c81635c2a270008::Jan 31 2017 07:00:05 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:00:09 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:00:15 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:00:19 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:00:25 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:00:29 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:00:35 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:00:39 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:00:45 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:00:49 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec

  c81635c2a270008::Jan 31 2017 07:00:55 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:00:59 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:01:05 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:01:09 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:01:15 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:01:19 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:01:25 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:01:29 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c1d635c2a270008::Jan 31 2017 07:01:29 GMT: INFO (info): (thr_info.c:5099) reads 0,0 : writes 0,0
  c1d635c2a270008::Jan 31 2017 07:01:29 GMT: INFO (info): (thr_info.c:5103) udf reads 0,0 : udf writes 0,0 : udf deletes 0,0 : lua errors 0

  c81635c2a270008::Jan 31 2017 07:01:35 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec
  c81635c2a270008::Jan 31 2017 07:01:35 GMT: INFO (info): (thr_info.c:5099) reads 0,0 : writes 0,0
  c81635c2a270008::Jan 31 2017 07:01:35 GMT: INFO (info): (thr_info.c:5103) udf reads 0,0 : udf writes 0,0 : udf deletes 0,0 : lua errors 0
  c1d635c2a270008::Jan 31 2017 07:01:39 GMT: INFO (info): (hist.c:137) histogram dump: reads (0 total) msec

```

### Count
The `count` command is used to display count of lines containing a match to the input pattern per interval. 
This command supports multiple input patterns, runtime server log selection, ignore-case, invert-match, uniq filter, and execution over specific time period. 

To avoid longer waiting time, we can use _-p_ option to set output page size. Its default value is 10, so after every 10 result entries it displays output.

```asciidoc
Log-analyzer> count -s "reads" -n "5,6" -f "Jan 31 2017 07:00:00" -d 100 -t 20
~~~~~~~~~~~~~~~~~~~~~~~~~cluster (Page-1)~~~~~~~~~~~~~~~~~~~~~~~~
NODE                :   c1d635c2a270008   |   c81635c2a270008   |   
Jan 31 2017 07:00:00:   2                 |   2                 |   
Jan 31 2017 07:00:20:   2                 |   2                 |   
Jan 31 2017 07:00:40:   2                 |   2                 |   
Jan 31 2017 07:01:00:   2                 |   2                 |   
Jan 31 2017 07:01:20:   4                 |   4                 |   
total               :   12                |   12                |   

```

### Diff
The `diff` command displays set of values and difference between consecutive values for search string in server logs.

It supports following five patterns of search string _S_ and values _Vi_ :
  * _S_    _V0_
  * _S_   (_V0_)
  * _S_    (_V0_, _V1_, …, _Vn_)
  * _S_    (_V0_
  * _V0_(_V1_)    _S_

This command supports runtime server log selection, ignore-case, output filter, and execution over specific time period. 

To avoid longer waiting time, we can use _-p_ option to set output page size. Its default value is 10, so after every 10 result entries it displays output.

 ```asciidoc
Log-analyzer> diff -s "fds - proto" -n "5,6" -f "Jan 31 2017 07:00:00" -d 100
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~fds - proto Diff (Page-1)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                :   c1d635c2a270008   .               |   c81635c2a270008   .               |   
.                   :   Total             Diff            |   Total             Diff            |   
Jan 31 2017 07:00:00:   [0, 510, 510]     [0, 510, 510]   |   [0, 508, 508]     [0, 508, 508]   |   
Jan 31 2017 07:00:10:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:00:20:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:00:30:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:00:40:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:00:50:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:01:00:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:01:10:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:01:20:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   
Jan 31 2017 07:01:30:   [0, 510, 510]     [0, 0, 0]       |   [0, 508, 508]     [0, 0, 0]       |   

```

### Histogram
The `histogram` command analyzes server logs and displays histogram measurements. It parses histogram lines for input histogram for provided time slices and calculates percentage of operations in each time slice.

We can get histogram latency for specific namespace by providing namespace name with option _-N_. This feature works only for logs of server version 3.9 and above.

To avoid longer waiting time, we can use _-p_ option to set output page size. Its default value is 10, so after every 10 result entries it displays output.

To get the information about histograms please see the [Histograms](/docs/operations/monitor/latency/index.html).

Example Output:
```asciidoc
Log-analyzer> histogram -h reads -f head -d 150 -p 20 -f 'Jan 27 2016 22:26:00'
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~reads Latency (Page-1)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
NODE                :   node0    .        .         .         |   node1    .        .         .         |   node2    .        .         .         |   
.                   :   % >1ms   % >8ms   % >64ms   ops/sec   |   % >1ms   % >8ms   % >64ms   ops/sec   |   % >1ms   % >8ms   % >64ms   ops/sec   |   
Jan 27 2016 22:26:10:   9.09     6.06     0.00      3.3       |   []       []       []        []        |   []       []       []        []        |   
Jan 27 2016 22:26:20:   0.00     0.00     0.00      0.8       |   3.70     0.00     0.00      2.7       |   11.11    0.00     0.00      0.9       |   
Jan 27 2016 22:26:30:   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.1       |   0.00     0.00     0.00      0.0       |   
Jan 27 2016 22:26:40:   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   
Jan 27 2016 22:26:50:   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   
Jan 27 2016 22:27:00:   0.00     0.00     0.00      0.6       |   0.00     0.00     0.00      0.7       |   0.00     0.00     0.00      0.4       |   
Jan 27 2016 22:27:10:   0.00     0.00     0.00      1.9       |   0.00     0.00     0.00      0.1       |   0.00     0.00     0.00      0.4       |   
Jan 27 2016 22:27:20:   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.7       |   0.00     0.00     0.00      0.6       |   
Jan 27 2016 22:27:30:   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   
Jan 27 2016 22:27:40:   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   
Jan 27 2016 22:27:50:   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.0       |   
Jan 27 2016 22:28:00:   0.00     0.00     0.00      0.2       |   0.00     0.00     0.00      0.2       |   0.00     0.00     0.00      0.0       |   
Jan 27 2016 22:28:10:   0.00     0.00     0.00      0.4       |   0.00     0.00     0.00      0.0       |   0.00     0.00     0.00      0.1       |   
Jan 27 2016 22:28:20:   0.00     0.00     0.00      1.7       |   0.00     0.00     0.00      0.2       |   0.00     0.00     0.00      0.1       |   
Jan 27 2016 22:28:30:   0.00     0.00     0.00      19.8      |   2.11     0.00     0.00      19.0      |   1.38     0.00     0.00      14.5      |   
avg                 :   0.61     0.40     0.00      1.0       |   0.42     0.00     0.00      1.0       |   0.89     0.00     0.00      1.0       |   
max                 :   9.09     6.06     0.00      19.8      |   3.70     0.00     0.00      19.0      |   11.11    0.00     0.00      14.5      |   
```

The following example shows `write` latency analysis for namespace `bar`.

```asciidoc
Log-analyzer> histogram -h write -N bar  -d 200 -f -4400
~~~~~~~~~~~~~~~~~~~~~~~bar - write Latency (Page-1)~~~~~~~~~~~~~~~~~~~~~~
NODE                :   bb9635c2a270008    .        .         .         |   
.                   :   % >1ms             % >8ms   % >64ms   ops/sec   |   
Sep 29 2016 08:33:30:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:33:40:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:33:50:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:34:00:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:34:10:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:34:20:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:34:30:   0.00               0.00     0.00      0.1       |   
Sep 29 2016 08:34:40:   0.00               0.00     0.00      0.1       |   
Sep 29 2016 08:34:50:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:35:00:   0.00               0.00     0.00      0.1       |   

~~~~~~~~~~~~~~~~~~~~~~~bar - write Latency (Page-2)~~~~~~~~~~~~~~~~~~~~~~
NODE                :   bb9635c2a270008    .        .         .         |   
.                   :   % >1ms             % >8ms   % >64ms   ops/sec   |   
Sep 29 2016 08:35:10:   0.00               0.00     0.00      0.2       |   
Sep 29 2016 08:35:20:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:35:30:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:35:40:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:35:50:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:36:00:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:36:10:   0.00               0.00     0.00      0.0       |   
Sep 29 2016 08:36:20:   0.00               0.00     0.00      0.0       |   
avg                 :   0.00               0.00     0.00      0.0       |   
max                 :   0.00               0.00     0.00      0.2       |
```

Also we can use raw histogram (with namespace name) instead of using `-N`, so above command can be written as follow.
```asciidoc
Log-analyzer> histogram -h {bar}-write  -d 200 -f -4400
```

### Pager
The `pager` command sets pager for output. For output which can not fit in
output console, this command gives option to scroll each output table vertically as well as horizontally.

