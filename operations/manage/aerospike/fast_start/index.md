---
title: Fast Restart
description: Learn how you can use the fast restart feature to enable your nodes to start up and re-join their clusters quickly on the Aerospike Enterprise Edition.
---

Aerospike Enterprise Edition has a feature enabling nodes to start up and
re-join their cluster much faster. A node with over 1 billion records will
restart in about 10 seconds (as opposed to 40+ minutes without this feature).
This allows cluster upgrades and various other operations to go much faster.

## How does it work?
This feature enables server nodes to make use of Linux system shared memory. A
server node’s primary index, along with various other critical data, is
allocated in system shared memory which persists even when the server process is
stopped. On restarting, the process re-attaches to the persisted memory
containing the index and other data, quickly rebuilds some internal state and
re-joins its cluster. It does not need to read all the record data from storage
drives to rebuild the index.

## What do I have to do to make this happen?
Nothing. For namespaces configured with flash storage or
[Data In Index](docs/reference/configuration/index.html#data-in-index), the
node’s default behavior is to always try to fast restart, so just start up
normally:

```
$ sudo systemctl start aerospike
```

You can check in the logs to see if the node performs a fast restart (also
referred to as WARM restart) for a namespace.

```
INFO (namespace): (namespace_ee.c:361) {test} beginning warm restart
```

## When does fast restart NOT happen?
There are some situations in which a server node will try to fast restart but
determines that it cannot do so, and will switch to a “cold restart”, where all
the record data is read from storage drives to rebuild the index:

- The namespace is configured as "data-in-memory". Since record data is not
  stored in system shared memory, all record data must be read to restore the data to
  memory. As of Enterprise Edition version 3.15.1.3, though, the index is not
  rebuilt, avoiding cases of deleted records being restored from storage in such
  cases (records not durably deleted while the node was not in the cluster would
  be brought back).
  
  With `data-in-memory true`, on server versions prior to 3.15.1.3,
  ```
  INFO (namespace): (namespace_ee.c:162) {test} can't warm restart if data-in-memory
  INFO (namespace): (namespace_ee.c:243) {test} beginning COLD start
  ```
  As of server version 3.15.1.3 and later,
  ```
  INFO (namespace): (namespace_ee.c:360) {test} beginning cool restart
  ```
- The storage drives were wiped clean while the server node was down. Very
  occasionally this is done, e.g. when repairing a drive with physical failures.
  Obviously if the record data was erased, the persisted index isn’t valid.
  Fortunately "cold restart" will be fast in this case — since there’s no data
  to read!
- The previous shutdown is not trusted. If a node stops unexpectedly, and we
  can’t be sure the persisted index is reliable and consistent with the stored
  data, it’s best to rebuild the index.
- The machine was rebooted. System shared memory survives the server process
  stopping, but not the machine rebooting.
- Prior to 3.13.0, if the order of namespaces in the config was changed.

If a node does switch to "cold restart", the log file will mention it, and
should indicate the reason for the switch.

{{#note}}
Note that when secondary indexes are defined, the start up time of a node can 
significantly slow down, depending on the number of secondary indexes defined
and their size. If a namespace meets the conditions to fast restart but has
secondary indexes defined, after the fast restart (primary index only), the
namespace will have to rebuild every secondary index defined, which would
require looking up record values from the persistent storage.
{{/note}}

See here for details on [Cold Restart](/docs/operations/manage/aerospike/cold_start).

## Can I force a “cold restart”?
Yes, although not necessary, you can manually force a “cold start” by using
the command:

```
sudo /etc/init.d/aerospike coldstart
```

## Can I monitor system shared memory used by Aerospike?
You can see the system’s shared memory blocks, using the command:

```
$ sudo ipcs -m
```

All the blocks listed that have keys starting with
```
0xae...
```

'ae' (as in Aerospike) are Aerospike shared memory blocks. For each namespace,
one or more 1GB blocks.  For example, for a node with two namespaces, you might
see:

```
------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      
0xae001100 0          root       666        1073741824 1                       
0xae002100 32769      root       666        1073741824 1                       
...
```

## Fast Restart for Data-In-Memory Namespace
If your data-in-memory namespace is a single-bin namespace with only numeric
data (integer or double), it can also take advantage of the fast-restart
feature.

To do so, add the "data-in-index" configuration to your namespace stanza:
 
```
namespace <namespace-name> {
...
    single-bin true
    data-in-index true
...
}
```

Then simply restart Aerospike, and fast restart should just happen!

If there is non-numeric data types in the database -- while reducing the index
for fast-restart, we'll check that each record is indeed an integer or double.
If we find a record that is not, the Aerospike process will exit with a message.
Once restarted successfully, any attempt to try writing non-numeric data, the
write will be rejected with error code 12 (INCOMPATIBLE_TYPE).
