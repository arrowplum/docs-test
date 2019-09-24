---
title: Configure - Log Rotate
description: Manage log rotations using the Linux tool, logrotate, with the Aerospike database.
---

{{#note}}
With journald, you do not need to explicitly setup log rotation as it is implicitly handled by the journaling subsystem when the logs are configured to be output to the console. Refer to the [systemd](/docs/operations/manage/aerospike/systemd) page for more details.
{{/note}}

{{#warn}}
When configured to log to multiple files, it is necessary to use the logrotate **sharedscripts** directive in order to preserve the content of all the file and prevent a race condition between the different SIGHUP signals. Such race may improperly close log sink file descriptors, which could lead to potential reuse of those file descriptors for client transactions. If that happens, the C client library, and therefore XDR, may crash upon receiving improper wire protocol messages for Aerospike versions 4.5.3.2 and earlier.
{{/warn}}

Aerospike uses the Linux tool `logrotate` to manage log rotations. Most versions of Linux should have logrotate already installed. If necessary, though, follow those steps to install it:

## Install logrotate binary

For CentOS:
```
sudo yum install logrotate
```

For Debian and Ubuntu:
```
sudo apt-get install logrotate
```

## Configure logrotate policy for aerospike

On Aerospike installation, a log rotation policy file is set up at `/etc/logrotate.d/aerospike` that specifies:
- The log should be rotated every day
- Compress the old log files
- Retain files for 90 days
- Use date as the suffix
- Use sharedscripts when rotating multiple files.

### Logrotate policy on non-systemd environment (requires an asd pid file)

```bash
/var/log/aerospike/aerospike.log {
    daily
    rotate 90
    dateext
    compress
    olddir /var/log/aerospike/
    sharedscripts
    postrotate
        /bin/kill -HUP `cat /var/run/aerospike/asd.pid`
    endscript
}
```

### Logrotate policy on systemd environments

If there are no pid files (on systemd based installations, if logrotate is preferred over journald), the postrotate kill -HUP command should use either `pidof asd` or `pgrep asd`.

```bash
/var/log/aerospike/aerospike.log {
    daily
    rotate 90
    dateext
    compress
    olddir /var/log/aerospike/
    sharedscripts
    postrotate
        /bin/kill -HUP `pidof asd`
    endscript
}
```

### Logrotate policy with multiple files.

Multiple log sync files can be configured to be rotated and use the same logrotate configuration and postrotate script.

```bash
/var/log/aerospike/aerospike.log /var/log/aerospike/xdr.log {
    daily
    rotate 90
    dateext
    compress
    olddir /var/log/aerospike/
    sharedscripts
    postrotate
        /bin/kill -HUP `pidof asd`
    endscript
}
```


{{#note}}
When typing the configuration lines above, there is a `<tab>` before **/bin/kill** preceeded by 4 spaces of indentation.
{{/note}}

### Force run a logrotate to test

```
sudo logrotate -f -v /etc/logrotate.d/aerospike

```

Example output:

```
logrotate -f -v /etc/logrotate.d/aerospike
reading config file /etc/logrotate.d/aerospike
olddir is now /var/log/aerospike/
Allocating hash table for state file, size 15360 B

Handling 1 logs

rotating pattern: /var/log/aerospike/aerospike.log  forced from command line (90 rotations)
olddir is /var/log/aerospike/, empty log files are rotated, old logs are removed
considering log /var/log/aerospike/aerospike.log
  log needs rotating
rotating log /var/log/aerospike/aerospike.log, log->rotateCount is 90
dateext suffix '-20171101'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
glob finding old rotated logs failed
renaming /var/log/aerospike/aerospike.log to /var/log/aerospike//aerospike.log-20171101
running postrotate script
compressing log with: /bin/gzip
```
