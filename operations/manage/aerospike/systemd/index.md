---
title: Aerospike systemd Daemon Management
description: Learn how you can start, coldstart, status, restart and stop the Aerospike daemon under systemd.
---

On certain Linux distributions based on [systemd](https://freedesktop.org/wiki/Software/systemd/)
(currently the Red Hat EL7-based family, including Red Hat Enterprise Linux 7, CentOS 7, Fedora 15+, Debian 8+, and Ubuntu 16.04+),
the Aerospike daemon and server logs are managed using the standard `systemd` tools,
[systemctl](https://freedesktop.org/software/systemd/man/systemctl.html)
and [journalctl](https://freedesktop.org/software/systemd/man/journalctl.html),
rather than via the Aerospike [SysV init script](/docs/operations/manage/aerospike).

The Aerospike daemon can be controlled using `systemctl`, which supports
the following commands:
- start
- status
- restart
- stop

### Start Aerospike Server
For Aerospike Enterprise, start instructs the server to
[Fast Restart](/docs/operations/manage/aerospike/fast_start), if running Aerospike Community or running a
namespace that isn't supported this command behaves the same as doing a [Cold Start](/docs/operations/manage/aerospike/cold_start).

{{#info markdown=true}}
For more information regarding restart modes, see [Restart Modes](/docs/operations/manage/aerospike/index.html#restart-modes).
{{/info}}

To start the Aerospike database service:
```bash
systemctl start aerospike
```

### Coldstart Aerospike Server
For Aerospike Enterprise, a cold start forces the server to rebuild the in-memory
index by scanning the persisted records. This may take a significant amount of
time depending on the size of the data. For Aerospike Community this is the same
behavior as `start`.

Since `systemctl` does not support application-specific commands, the
script `asd-coldstart` is available for this purpose:
```bash
asd-coldstart
```

### Get Running Status of Aerospike Server
To determine if the Aerospike daemon is currently running, use the `status`
command:
```bash
systemctl status aerospike
```

{{#info markdown=true}}
The status command doesn't inform you when the service port is ready, instead
use the **STATUS** info command which will return "OK" when ready:
```
asinfo -v STATUS
```
{{/info}}

### Restart Aerospike Server
The `restart` command is equivalent to running `stop` followed by `start`:
```bash
systemctl restart aerospike
```

### Stop Aerospike Server
To shut down the Aerospike Server use the `stop` command:
```bash
systemctl stop aerospike
```

{{#note}}
**Note**: Aerospike flushes the data in the buffers to the disk (if data is configured to be persisted on device) when Aerospike is stopped gracefully. However, in other situations (unexpected loss of power, process crash), the data present in the buffer may not make it to the device. Single node crashes with a replication-factor of 2 will not cause any data loss, though. For multiple nodes crashing simultaneously, different configurations can be used to avoid any data loss, including, for example, [rack aware](/docs/operations/configure/network/rack-aware/index.html) and [`commit-to-device`](/docs/reference/configuration/#commit-to-device) (available on [strong-consistency](/docs/reference/configuration/#strong-consistency) namespaces in versions 4.0 and above).
{{/note}}

---

### Managing Server Logs under `systemd`

Under `systemd`, the [journald](https://freedesktop.org/software/systemd/man/systemd-journald.service.html)
is the standard facility for managing logs for all Linux daemon processes uniformly.
A "structured, indexed centralized journals" is supposed to be better than the old fashioned way of text-based log files.

The Aerospike log can be accessed via the `journalctl` command.

```bash
journalctl -u aerospike -a -o cat -f
```

- The `-a` option ensures nothing is suppressed from the Aerospike log messages,
even if some lines are very long.
- The `-o cat` option presents the raw Aerospike log without prepending
the journal's timestamp and other metadata to each line.
- The `-f` option will "follow" the log as it is generated.

{{#info markdown=true}}
When running under `systemd`, it is still possible to direct the
Aerospike Server to create (and rotate) log files. For more information
on managing Aerospike Server log files, please see [Log Files](/docs/operations/manage/log).
{{/info}}

### Using `asloglatency` under `systemd`

To use the `asloglatency` tool under `systemd`, first extract the
desired portion of the log into a temporary file using `journalctl`, e.g.:

```bash
journalctl -u aerospike -a -o cat --since "2016-03-17" --until "2016-03-18" | grep GMT > /tmp/aerospike.20160317.log
```

then run `asloglatency` on the extracted log file, e.g.:

```bash
asloglatency -h writes_master -f head -l /tmp/aerospike.20160317.log
```

### Changing user and group under `systemd`

{{#warn}}If upgrading to Aerospike Server 4.5.3.2 or newer, the modifications to /etc/systemd/system/aerospike.service.d/user.conf step below should not be configured.{{/warn}}

For **RHEL 7, Ubuntu 16.04+, Debian 8+** (distributions based on systemd) running versions prior to 4.5.3.2, the following additional steps are required (besides the steps described on the [configuring as non root](/docs/operations/configure/non_root) page):
```
cat > /etc/systemd/system/aerospike.service.d/user.conf <<EOF
[Service]
User=AerospikeUser
Group=AerospikeGroup
EOF
```
Refer to this Knowledge-Base: https://discuss.aerospike.com/t/changing-usergroup-of-asd-process-under-systemd/4734 article for further details.
