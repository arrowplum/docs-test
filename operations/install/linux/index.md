---
title: Install Aerospike on Linux
description: Aerospike Server comes packaged as .rpm, .deb, binary and source tarballs.
---
Aerospike Server is developed and optimized for 64-bit Linux, and comes packaged
as `.rpm`, `.deb`, binary and source tarballs.

### [Install on Red Hat](/docs/operations/install/linux/el6)
Install Aerospike Server on Red Hat Enterprise, CentOS, Fedora, Amazon Linux,
Oracle Linux, and other Linux distributions using [`.rpm`](http://rpm.org/) packages.

### [Install on Ubuntu](/docs/operations/install/linux/ubuntu)
Install Aerospike Server on Ubuntu Linux using `.deb` packages.

### [Install on Debian](/docs/operations/install/linux/debian)
Install Aerospike Server on Debian Linux using `.deb` packages.

### [Install using Binary Package](/docs/operations/install/linux/other)
Install Aerospike Server as a precompiled binary for Linux.

### Building from Source
Build Aerospike from source on a 64-bit GNU/Linux system.
Source code and instructions are available at the
[aerospike/aerospike-server](https://github.com/aerospike/aerospike-server)
repository on Github.

{{#note}}
**Note** - It is not necessary to use homogenous versions or distributions of Linux for all nodes within a cluster.  All nodes should run on supported distributions or versions.  Running on heterogenous OS versions or distributions could make troubleshooting a performance issue more complex due to an increased number of variables to consider.
{{/note}}
