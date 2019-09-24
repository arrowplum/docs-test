---
title: Running Aerospike Daemon - command line options
description: Learn how to manage Aerospike daemon using command line options
---

In some circumstances, especially Docker, you'll want and need to run Aerospike from the command line. For docker, in particular, the service subsystem may or may not work the way you expect, and you should be running foreground within a container.

The command to start Aerospike ( without a service manager ) is '`asd`'. If you build from source, this is in the '`target/Linux-x86_64/bin`' directory. If you install via the package manager, it is '`/usr/bin/asd`' ( and thus, usually, in the path ).

If you run '`asd --help`' you'll get a list of the commands. As of Aerospike 3.10.1, here is that list of options:

- `--version`

Print edition and build version information and exit.

The command line options are:

- `--config-file` 

Specify the location of the Aerospike server config file. If this option is not
specified, the default location `/etc/aerospike/aerospike.conf` is used.

- `--foreground`

Specify that Aerospike not be daemonized. This is useful for running Aerospike
in gdb. Alternatively, add '`run-as-daemon false`' in the service context of the
Aerospike config file. Do not use if you are running Docker.

- `--fgdaemon`

Specify that Aerospike is to be run as a "new-style" (foreground) daemon. This
is useful for running Aerospike under systemd or Docker. asd runs in the foreground and ignores the following configuration items:
user ('user'), group ('group') and PID file ('pidfile').
Further information about New Style Daemons can be found at: (*) http://0pointer.de/public/systemd-man/daemon.html#New-Style%20Daemons

- `--cold-start`

(Enterprise edition only.) At startup, force the Aerospike server to read all
records from storage devices to rebuild the index, rebuilding any possible shared memory segments that might have existed in previous boots.

- `--instance <0-15>`

(Enterprise edition only.) In order to run multiple copies of Aerospike, and persist indexes in DRAM, this "instance ID" must be used. It is used to create different sets of shared memory segments, and to find a previous launches' shared memory segment. To run multiple copies of asd, you will also need to have a separate "working directory" for each instance specified in the config file - those can't be shared.
