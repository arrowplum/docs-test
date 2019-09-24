---
title: Install on RedHat and CentOS
description: This tutorial covers installing Aerospike Tools on RedHat and CentOs6+ system
---
```bash
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/el6'
# for CentOs 7, replace "el6" with el7
tar -xvf aerospike-tools.tgz
cd aerospike-tools-*.el6
# for CentOs 7, replace el6 with el7
sudo ./asinstall # will install the .deb packages
asadm --version # will show the installed version of asadm
asbackup --version # will show the installed version of asbackup
asrestore --version # will show the installed version of asrestore
aql --version # will show the installed version of aql
```
### Overview
Use this tutorial to install Aerospike Tools on an RedHat and CentOs systems using `.rpm` packages.

Above is a condensed version, and below are the step-by-step instructions.

{{md (path-resolve (path-dirname page.src) "download.part.md") }}

{{md (path-resolve (path-dirname page.src) "install.part.md") }}

{{md (path-resolve (path-dirname page.src) "validate.part.md") }}
