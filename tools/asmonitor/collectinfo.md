---
title: asmonitor - collectinfo
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

The "collectinfo" command gathers and packages system environment logs and stats frequently requested when assessing cluster health.

Because a portion of the data often requires root access, run "asmonitor" with "sudo" or within a root shell to generate a full report.

To run on a node running aerospike server, you can run it as:
```
sudo asmonitor -e 'collectinfo'
```
If there is no aerospike server running at the time of collecting the data, then you should run it with ```force``` argument to collect the rest of system metrics/log data.
```
sudo asmonitor -e 'collectinfo force'
```
 

Upon completion, collectinfo generates a file containing the collection stats and a compressed copy.

If you have a question for support, including the compressed copy with the request is greatly appreciated.
Data collected by collectinfo

See below for an itemized list of environment logs and stats gathered.
System related:

     uname -a
     lsb_release -a
     ls /etc|grep release|xargs -I f cat /etc/f
     rpm -qa|grep -E "citrus|aero"
     dpkg -l|grep -E "citrus|aero"
     tail -n 1000 /var/log/aerospike/*.log
     tail -n 1000 /var/log/citrusleaf.log
     tail -n 1000 /var/log/clxdr.log
     netstat -pant|grep 3000
     top -n3 -b
     free -m
     df -h
     ls /sys/block/{sd*,xvd*}/queue/rotational |xargs -I f sh -c "echo f; cat f;"
     lsof
     dmesg
     iostat -x 1 10
     vmstat -s
     vmstat -m
     iptables -L
     cat /etc/aerospike/aerospike.conf
     cat /etc/citrusleaf/citrusleaf.conf
     sudo lsof|grep `sudo ps aux|grep -v grep|grep -E 'asd|cld'|awk '{print $2}'


Aerospike related:

     Node
     Namespace
     XDR
     SETS
     printconfig
     compareconfig
     latency
     stat
     objsz
     ttl
     evict

For a description of Aerospike related statistics collected by "collectinfo" see asmonitor documentation

 {{#info}}
collectinfo is available in tools version 3.0.30 onwards.
{{/info}}
