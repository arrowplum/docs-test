---
title: asmonitor - Commands
description: Learn about the 14 commands such as help, info, stat, latency, add, remove, etc.
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
 
### help

Display usage information
```bash
Monitor> help
```

### info

Display the tables of information for: Node, Namespace, Sets, and XDR for the cluster or node(s) specified by the -h parameter. You may also configure custom tables to display as well.
```bash
Monitor> info [<table>] [-h <host>[:<port>][, ...]] [-p <port>]
```

The `<table>` argument specifies the table to display. If not specified, then all tables will be displayed. Tapping [TAB] twice after info will display the list of available tables. Default tables available are Node, Namespace and XDR.

The -h option specifies a comma-separated list of hostnames or IP addresses to inquire information from. The values for the -h option are: 

- The `<host>` parameter is the hostname of the server. 
- The `<port>` parameter is optional, and will default to the port specified by the -p option if not specified.

The -p option specifies the default port for the hosts being inquired, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.

### asinfo

Read or modify configuration values in the cluster or on node(s) specified by the -h parameter.
```bash
Monitor> asinfo [-v <value>] [-h <host>[:<port>][, ...]] [-p <port>]
```

The -v option specifies the information to inquire from the nodes. Running a command using asinfo inside asmonitor runs it on all the nodes in the cluster, unless an explicit -h is passed to the asinfo command as well.

The -h option specifies a comma-separated list of hostnames or IP addresses to inquire information from. The values for the -h option are: 

- The `<host>` parameter is the hostname of the server. 
- The `<port>` parameter is optional, and will default to the port specified by the -p option if not specified.

The -p option specifies the default port for the hosts being inquired, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.


### stat

Get the statistics for the cluster or for node(s) specified by the -h parameter.
```bash
Monitor> stat [-v <value>] [-h <host>[:<port>][, ...]] [-p <port>]
```

The -v option specifies the stat to inquire from the nodes.

The -h option specifies a comma-separated list of hostnames or IP addresses to inquire stats from. The values for the -h option are: 

- The `<host>` parameter is the hostname of the server. 
- The `<port>` parameter is optional, and will default to the port specified by the -p option if not specified.

The -p option specifies the default port for the hosts being inquired, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.

### compareconfig

Compare the server configuration against the rest of the cluster and display parameters that do not match.  Does not compare namespace configuration.
```bash
Monitor> compareconfig [-h <host>[:<port>][, ...]] [-p <port>]
```

The -h option specifies a comma-separated list of hostnames or IP addresses to inquire information from. The values for the -h option are: 

- The <host> parameter is the hostname of the server. 
- The <port> parameter is optional, and will default to the port specified by the -p option if not specified.

The -p option specifies the default port for the hosts being inquired, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.

### checkmem

Check if the total configured memory across namespaces on a node is less than the installed physical memory on the node. 
```
Monitor> checkmem
```

### latency

Display the latency histogram data for the cluster or node(s) specified by the -h parameter.
```bash
Monitor> latency [-h <host>[:<port>][, ...]] [-p <port>]
                 [-v <filter>] [-k (reads | writes_master | writes | writes_reply | proxy)]
                 [-d <seconds>] [-b <seconds>] [-s <seconds>]
                 [-t] [-m] [-c]
```

The -h option specifies a comma-separated list of hostnames or IP addresses to inquire information from. The values for the -h option are: 

- The <host> parameter is the hostname of the server. 
- The <port> parameter is optional, and will default to the port specified by the -p option if not specified.

The -p option specifies the default port for the hosts being inquired, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.

The -v <histogram> option specifies the filter on the histrogram names to  display. This may be a wildcard value specified via an asterisk "*", e.g. *expires*.

The -k <type> option specifies the histogram type to filter on. The possible values are: "reads", "writes_master", "writes", "writes_reply" and "proxy".

The -d <seconds> option specifies the duration for polling the histogram information.

The -b <seconds> option specifies the number of seconds before "now" to gather information on.

The -s <seconds> option specifies the interval in seconds to analyze.

The -t option will display throughput data.

The -m option will group the output by node.

The -c option will run the histogram configuration.

### watch

Continuously watch a value and update the display.
```bash
Monitor> watch -n <seconds> <command>
```

The command argument is the name of the command to poll.

The -n <seconds> is the polling interval.

### add

Add one or many nodes to the current list of monitored nodes.
```bash 
Monitor> add -h <host>[:<port>][, <host>[:<port>] [, ...]] [-p <port>]
```

The -h option specifies a comma-separated list of hostnames or IP addresses to added to the list of monitored nodes. The values for the -h option are:

- The `<host>` parameter is the hostname of the server. 
- The `<port>` parameter is optional, and will default to the port specified by the -p option if not specified.

The -p option specifies the default port for the hosts being added, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.

### remove

Remove a node(s) from the list of monitored nodes.
```bash
Monitor> remove -h <host>[:<port>][,...] [-p <port>]
```

The -h option specifies a comma-separated list of hostnames or IP addresses to remove from the list of monitored nodes. The values for the -h option are: 

- The `<host>` parameter is the hostname of the server. 
- The `<port>` parameter is optional, and will default to the port specified by the -p option if not specified.

The -p option specifies the default port for the hosts being removed, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.

### edittable

Edits and existing table.
```bash 
Monitor> edittable
```

### sorttable

Sorts a table by column.
```bash
Monitor> sorttable
```

### restoredefault

Restore table defaults.
```bash
Monitor> restoredefaults
```


### exit

Exit the console
```bash
Monitor> exit
```
