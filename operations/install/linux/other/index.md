---
title: Install using Binary Package
description: This tutorial covers installing Aerospike as a binary for Linux systems without requiring root priviledges.
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 7
---
<style>
ol.steps {
  padding-top: 1em;
}
</style>

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/tgz'
tar -xvf aerospike.tgz
cd aerospike-server
./bin/aerospike init
sudo ./bin/aerospike start && \
sudo tail -f ./var/log/aerospike/aerospike.log | grep cake
# Wait for it. "service ready: soon there will be cake!"
# For systemd based installations, check in the journald facility.
```

### Overview

Use this tutorial to install Aerospike on any Linux system with glibc 2.11 or newer. glibc 2.11 was released in January, 2009 and most modern Linux distributions - Debian 6+, Centos 6+, OpenSUSE 12+, Ubuntu 10.04+ support this this version.

{{#info}}
This package contains an Aerospike binary that can be run in a user directory without installing
software, and does not require root privileges.
{{/info}}

{{md (path-resolve (path-dirname page.src) "download.part.md") }}

{{md (path-resolve (path-dirname page.src) "install.part.md") }}

{{md (path-resolve (path-dirname page.src) "run.part.md") }}

{{md (path-resolve (path-dirname page.src) "../../common/nextsteps.part.md") }}

