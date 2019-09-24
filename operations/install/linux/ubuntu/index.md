---
title: Install on Ubuntu
description: This tutorial covers installing Aerospike an Ubuntu system
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 6
---
<style>
ol.steps {
  padding-top: 1em;
}
</style>

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/ubuntu18'
# for ubuntu 16.04, replace "ubuntu18" with ubuntu16
# for ubuntu 14.04, replace "ubuntu18" with ubuntu14
tar -xvf aerospike.tgz
cd aerospike-server-community-*-ubuntu18*
# for ubuntu 16.04, replace "ubuntu18" with ubuntu16
# for ubuntu 14.04, replace "ubuntu18" with ubuntu14
sudo ./asinstall # will install the .deb packages
sudo service aerospike start && \
sudo tail -f /var/log/aerospike/aerospike.log | grep cake
# Wait for it. "service ready: soon there will be cake!"
# For systemd based installations, check in the journald facility.
```

### Overview
Use this tutorial to install Aerospike on an Ubuntu system using `.deb` packages.

Above is a condensed version, and below follow the step-by-step instructions.

{{#note markdown=true}}
Aerospike will only run on Ubuntu 14.04 LTS or newer. For **systemd platforms** 
such as Ubuntu 16.04, follow the installation steps below but refer to [systemd](/docs/operations/manage/aerospike/systemd) 
for best practices on running and managing Aerospike under systemd.
{{/note}}

{{#info}}
Note that root access (`sudo`) is required to install packages. If you do not 
have root access, install the [binary distribution](/docs/operations/install/linux/other).
{{/info}}

{{#info}}
Aerospike's Community Edition is configured to **transmit anonymous usage statistics**.
We ask your help in making Aerospike better by leaving this feature enabled.
You can learn about our goals, how we use the data, and how to disable the feature [here](/aerospike-telemetry).
{{/info}}


{{md (path-resolve (path-dirname page.src) "download.part.md") }}

{{md (path-resolve (path-dirname page.src) "install.part.md") }}

{{md (path-resolve (path-dirname page.src) "run.part.md") }}

{{md (path-resolve (path-dirname page.src) "../../common/nextsteps.part.md") }}

