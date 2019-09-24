---
title: Install Aerospike on Red Hat Variants
description: This tutorial covers installing Aerospike on Red Hat Enterprise Linux, CentOS Linux and similar distros.
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
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/el6'
tar -xvf aerospike.tgz
cd aerospike-server-community-*-el6
sudo ./asinstall # will install the .rpm packages
sudo service aerospike start && \
sudo tail -f /var/log/aerospike/aerospike.log | grep cake
# Wait for it... "service ready: soon there will be cake!"
# For systemd based installations, check in the journald facility.
```

### Overview
Use this tutorial to install Aerospike on Red Hat Enterprise, CentOS, Fedora,
Amazon Linux, Oracle Linux, and other Linux distributions using `.rpm` packages.

Above is a condensed version, and below follow the step-by-step instructions.

{{#warn}}
The CentOS 7 RPM packages are built with **OpenSSL 1.0.2**, which is the current shipping package version. If you are running on an older version of CentOS 7 which uses OpenSSL 1.0.1, the installation will not succeed due to a dependency mismatch. Please resolve by either updating CentOS patches (security or whole release) to a point where OpenSSL 1.0.2 is used, or install OpenSSL 1.0.2 using `yum update openssl`.
{{/warn}}

{{#note markdown=true}}
For **systemd platforms** such as CentOS 7, follow the installation steps below but refer to [systemd](/docs/operations/manage/aerospike/systemd) 
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

