---
title: Install on Debian
description: This tutorial covers installing Aerospike on a Debian system.
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
# for Debian 10:
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/debian10'
# for Debian 9:
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/debian9'
# for Debian 8:
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/debian8'
tar -xvf aerospike.tgz
cd aerospike-server-community-*-debian?
sudo ./asinstall # will install the .deb packages
sudo service aerospike start && \
sudo tail -f /var/log/aerospike/aerospike.log | grep cake
# Wait for it. "service ready: soon there will be cake!"
# For systemd based installations, check in the journald facility.
```

### Overview

Use this tutorial to install Aerospike on a Debian system using .deb packages.

Above is a condensed version, and below follow the step-by-step instructions.

{{#note markdown=true}}
For Debian systems, Aerospike will only run on Debian 6 or later. For **systemd platforms** 
such as Debian 8, 9, or 10, follow the installation steps below but refer to [systemd](/docs/operations/manage/aerospike/systemd) 
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

