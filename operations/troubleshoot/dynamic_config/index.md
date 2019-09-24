---
title: How to ..?
description: How to ..?
---

### Dynamically change parameters/configuration

You can use `asinfo` to set various parameters while server is running. Please be aware that these are transient changes and will go away on server restart.
You should add the changes to your configuration file if you want the changes to persist across server restarts.

{{#note}}
Most of our Client APIs support using the info commands natively by calling the appropriate API with
the desired value string. You can find a list of value strings following the '-v' in the commands that follow.
{{/note}}

### Use asinfo to Write Parameters Dynamically

To set a value dynamically with asinfo:

`asinfo -h [host ip] -p [port] -v [set-config command string]`

Here is an example on how to change a variable on the service stanza:

`$ asinfo -v "set-config:context=service;proto-fd-max=50000"`

Here is an example on how to change a variable on the network.heartbeat stanza:

`$ asinfo -v "set-config:context=network;heartbeat.interval=10"`

Here is an example on how to change a variable “high-water-memory-pct” on the namespace “test”:

`$ asinfo -v "set-config:context=namespace;id=test;high-water-memory-pct=50"`

#### Set common parameter settings

{{#warn}}WARNING: The examples below are meant to illustrate dynamic configuration changes using asinfo. You should not use the commands below without understanding the requirement. A wrong setting value could further aggravate a minor problem in a running cluster. {{/warn}}

### Increase the memory setting of a namespace.

Aerospike memory can be increased/decreased dynamically, but it cannot be decreased to less than half the current value.

`set-config:context=namespace;id=<NAMESPACE>;memory-size=3G;`

or

`set-config:context=namespace;id=<NAMESPACE>;memory-size=3221225472;`


### Slow down nsup/evictor/expirer

Set the number of seconds after which the nsup thread will wake up, evict and expire objects.

`set-config:context=service;nsup-period=3600;`

### Slow down secondary index garbage collector

Set the number of seconds the sindex gc thread will wake up and garbage collect.

`set-config:context=service;sindex-gc-period=3600;`

Set maximum number of secondary index entries sindex gc processes every second.

`set-config:context=service;sindex-gc-max-rate=100000;`


### Switch on microbenchmark

In order to investigate complex issues/slow transactions, sometimes it’s helpful to enable microbenchmarks and storage-microbenchmarks , which will print additional histograms in the logs.

See [Histograms from Aerospike Logs](/docs/operations/monitor/latency) for more details.


### Alter the speed of migrations

Tweak global migration parameters:

See [Tuning Migrations](/docs/operations/manage/migration)

### Tune the Defragmentation process

See Knowledge-Base on [Defragmentation](https://discuss.aerospike.com/t/defragmentation/718)

### Change the logging level for a particular component

Change the logging threshold of any component. The syntax below should be followed. Allowed log levels are debug, detail, warning, info.

```
log-set:id=0;migrate=debug;
log-set:id=0;nsup=debug;
log-set:id=0;rw=debug
```

You can find all the available logging parameters by using ` asinfo -v log/0`

The resultant output shows the logging component and the present value for that component.

```
$ asinfo -v log/0
requested value  log/0
value is  cf:misc:INFO;cf:alloc:INFO;...
```

For more on this, please check the [asinfo](/docs/tools/asinfo) docs.

### Change the high-water-mark for memory

Change the high watermark for memory or disk. If the used memory/disk percent increases this value, the objects will be [evicted](/docs/operations/configure/namespace/retention) from the database.

```
set-config:context=namespace;id=usermap;high-water-memory-pct=45
set-config:context=namespace;id=test;high-water-disk-pct=45
```



