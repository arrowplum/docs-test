---
title: Working with the Log File
description: Learn how to work with the Aerospike log files so that they do not grow too large. 
---

{{#todo markdown=true}}
- Refer to log line reference page.
{{/todo}}

Logging settings are normally configured during installation to use [logrotate](/docs/operations/configure/log/logrotate.html) so that log files do not grow too large.

### Rolling the Log File Manually

Follow these steps to manually roll the log file: 

1. Link the existing log file to a new file:

  ```bash
  $ sudo link /var/log/aerospike/aerospike.log [file path]
  ```

2. Find the server's process ID:
  
  ```
  $ cat pidfile /var/run/asd.pid
  ```

3. Send SIGHUP to the server (the asd process):
  
  ```
  $ sudo kill -s SIGHUP [asd process id]
  ```

  In the log file you will see a message saying:

  ```
  Signal HUP received, rolling log
  ```

The `/var/log/aerospike/aerospike.log` will be truncated and new messages will be added to the existing file. The old file will be in the location specified by the `link` command in step #1.

### Determining the Log ID and the Location of the Log File

Occasionally, it might be necessary to change the log levels of different units during cluster use. Aerospike supports changing these parameters dynamically though scripted API calls using asinfo.  Changing these settings does not require a server or cluster restart.

To determine where the log file is located use the command:

```bash
$ asinfo -h 127.0.0.1 -p 3000 -v "logs"
```

This command returns the logs that are configured and their IDs and the output looks something like this:

```bash
value is  0:/var/log/aerospike/aerospike.log
```

At the present, only one log is used, log 0. For all logging commands, specify log 0.

### Changing the Location of the Log File

To change the default location of the log file, you must modify the [configuration settings](/docs/operations/configure/log).

### Changing Logging Levels

The log levels are described in the logging overview.

To determine the current logging levels use the command:

```bash
$ asinfo -h 127.0.0.1 -p 3000 -v "log/0"
```

At the present, only one log is used, log 0. In the command above, log/0 refers to log ID 0.

This command returns the log contexts and their current levels:

```
value is  cf:misc:INFO;cf:alloc:INFO;cf:hash:INFO;cf:rchash:INFO;cf:shash:INFO;cf:queue:INFO;cf:msg:INFO;cf:redblack:INFO;cf:socket:INFO;cf:timer:INFO;cf:ll:INFO;cf:arenah:INFO;cf:arena:INFO;config:INFO;namespace:INFO;as:INFO;bin:INFO;record:INFO;proto:INFO;particle:INFO;demarshal:INFO;write:INFO;rw:INFO;tsvc:INFO;test:INFO;nsup:INFO;proxy:INFO;hb:INFO;fabric:INFO;partition:INFO;paxos:INFO;migrate:INFO;info:INFO;info-port:INFO;storage:INFO;drv_mem:INFO;drv_fs:INFO;drv_files:INFO;drv_ssd:INFO;drv_kv:INFO;scan:INFO;index:INFO;batch:INFO;trial:INFO;xdr:INFO;cf:rbuffer:INFO;fb_health:INFO
```

Typically all contexts are set to INFO to minimize excess logging.  Contexts can also be set to DEBUG or WARNING.

To change the logging level use a command like the one below.  This example demonstrates how to upgrade the log level for the migrate context from INFO to DEBUG:

```bash
$ asinfo -h 127.0.0.1 -p 3000 -v "log-set:id=0;migrate=debug"
```

where the id is the log ID as described above (currently 0).  This command changes the logging level until the server restarts or until you change the log level again.

This example demonstrates how to change the log level for the migrate context back to INFO:

```bash
$ asinfo -h 127.0.0.1 -p 3000 -v "log-set:id=0;migrate=info"
```
