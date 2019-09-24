---
title: Configure - Log
description: Learn how to set the logging context in the configuration file to specify the location and desired log level within Aerospike.
---

The logging context in the configuration file specifies the location of the Aerospike log as well as the logging level desired.
Below is an example of a logging context:

```bash
logging {
    file /var/log/aerospike/aerospike.log {
        context any info
    }
}
```
The above example tells the node to log to `/var/log/aerospike/aerospike.log`, at info level.

For systemd based system, the logs by default are configured to go to the console:
```bash
logging {
    console { # systemd based 
        context any info
    }
}
```
Please see [systemd](/docs/operations/manage/aerospike/systemd) for more details.

{{#note}}
Make sure the directory exists - if the log directory does not exist, Aerospike will NOT create this directory automatically and the server will not start.
{{/note}}

Every message produced by the server has a context (which names the part of the code that generated the message), and a severity level (which describes how acute the message is). The severity levels are as follows:

| Severity | Meaning                       |
|----------|-------------------------------|
| critical | Critical error messages       |
| warning  | Warning messages              |
| info     | Informational messages        |
| debug    | Debugging information         |
| detail   | Verbose debugging information |


It is possible to change the logging level for all context, or only for a specific context of choice.

The example below logs all context at info level, and in addition, the "migrate" context at "debug" level.

```bash
logging {
    file /var/log/aerospike/aerospike.log {
        context any info
        context migrate debug
    }
 }
```
You can find out all available context using `asinfo`, via the [log](/docs/reference/info#log) command.

It is also possible to change the logging level dynamically on the node by using `asinfo`. 
Please see [log-set](/docs/reference/info#log-set).

### Find existing log location

For finding the logs location of a running instance, you can either check the config file or run

```bash
~$ asinfo -v logs
     requested value  logs
     value is  0:/var/log/aerospike/aerospike.log
```
<section id="logrotate"></section>

### Log Rotate

Refer to [logrotate](/docs/operations/configure/log/logrotate.html).
