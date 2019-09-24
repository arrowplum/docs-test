---
title: Aerospike Daemon Management
description: Learn how you can start, coldstart, status, restart and stop the Aerospike daemon using the SysV init script with the Aerospike database.
---

The Aerospike daemon can be controlled using the SysV init script located at
`/etc/init.d/aerospike` which supports the following options:
- start
- coldstart
- status
- restart
- stop

### Start Aerospike Server
For Aerospike Enterprise, start instructs the server to
[Fast Restart](/docs/operations/manage/aerospike/fast_start), if running Aerospike Community or running a
namespace that isn't supported this command behaves the same as [Cold Start](/docs/operations/manage/aerospike/cold_start).
For more information regarding restart modes, see [Restart Modes](/docs/operations/manage/aerospike/index.html#restart-modes).

To start the Aerospike database service:
```bash
/etc/init.d/aerospike start
OR
sudo service aerospike start
```

### Coldstart Aerospike Server
For Aerospike Enterprise, coldstart forces the server to rebuild the in-memory
index by scanning the persisted records. This may take a significant amount of
time depending on the size of the data. For Aerospike Community this is the same
behavior as `start`.
```bash
/etc/init.d/aerospike coldstart
OR
sudo service aerospike coldstart
```

See here for details on [Cold Restart](/docs/operations/manage/aerospike/cold_start).

### Get Running Status of Aerospike Server
To determine if the Aerospike daemon is currently running, use the `status`
command:
```bash
/etc/init.d/aerospike status
OR
sudo service aerospike status
```

{{#info markdown=true}}
The status command doesn't inform you when the service port is ready, instead
use the **STATUS** info command which will return "OK" when ready:
```
asinfo -v STATUS
```
{{/info}}
{{#todo markdown=true}}
- The above node made me realize that the init script should do something similar.
  - See AER-2468
{{/todo}}

### Restart Aerospike Server
The `restart` command is equivalent to running `stop` followed by `start`:
```bash
/etc/init.d/aerospike restart
OR
sudo service aerospike restart
```

### Stop Aerospike Server
To shut down the Aerospike Server use the `stop` command:
```bash
/etc/init.d/aerospike stop
OR
sudo service aerospike stop
```

{{#note}}
**Note**: Aerospike flushes the data in the buffers to the disk (if data is configured to be persisted on device) when Aerospike is stopped gracefully. However, in other situations (unexpected loss of power, process crash), the data present in the buffer may not make it to the device. Single node crashes with a replication-factor of 2 will not cause any data loss, though. For multiple nodes crashing simultaneously, different configurations can be used to avoid any data loss, including, for example, [rack aware](/docs/operations/configure/network/rack-aware/index.html) and [`commit-to-device`](/docs/reference/configuration/#commit-to-device) (available on [strong-consistency](/docs/reference/configuration/#strong-consistency) namespaces in versions 4.0 and above).
{{/note}}

---

### Restart Modes
Whenever the server is restarted gracefully, typically for a maintenance event,
the namespaces may resynchronize in 3 ways depending on the namespace
configuration and Aerospike version:
- Resynchronize from other nodes.
- Resynchronize from persisted storage and other nodes.
- Resynchronize from memory using [Fast Restart](/docs/operations/manage/aerospike/fast_start).

All resynchronization modes are performed automatically on start without
operator intervention.

{{#info}}
To maintain performance during the synchronization process (migrations), new
database operation requests are handled with a higher priority. If the server is
down for a long time, then the re-synchronization (migrations) may take a
significant amount of time, possibly many hours depending on current activity
levels, the size of the data and the speed of the network.
{{/info}}

{{#note}}
Migrations can be [tuned to go faster](/docs/operations/manage/migration/index.html#speeding-up-the-migration-rate).
{{/note}}

#### Restart Mode for In-Memory Without Persistence

In-memory databases without persistence always have to re-synchronize data from
the other nodes, assuming the cluster is configured with a
[replication-factor parameter]({{book.baseurl}}/operations/configure/namespace#replication-config) greater
than 1. If the replication-factor is set to 1 then the unreplicated data
previously stored in this namespace will be lost.

#### Restart Mode for In-Memory with Persistence

As of version 3.15.1.3, in-memory databases running the Enterprise Edition will persist their primary index in shared memory. 
The data will still be reloaded from storage. Once the
data is reloaded, the server will automatically re-synchronize with the other
servers in the cluster to get newer data updates.

#### Restart Modes for Flash Storage (and Data In Index)

Databases with Flash (SSD) storage have the same mode as for in-memory with
persistence (described above). In Aerospike Enterprise Edition, a fast restart
mode is used: [Fast Restart](/docs/operations/manage/aerospike/fast_start).

{{#info}}
In case of ungraceful shutdown of Aerospike or hardware restart,
[Fast Restart](/docs/operations/manage/aerospike/fast_start)
will fall back to regular restart [Cold Restart](/docs/operations/manage/aerospike/cold_start), reading data from Flash
storage to rebuild the index.
{{/info}}
