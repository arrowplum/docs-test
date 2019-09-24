---
title: Install on Debian
description: This tutorial covers installing Aerospike Tools on Debain system
---
```bash
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/debian10'
# for Debian 9, replace "debian10" with debian9
# for Debian 8, replace "debian10" with debian8
tar -xvf aerospike-tools.tgz
cd aerospike-tools-*-debian10/
# for Debian 9, replace "debian10" with debian9
# for Debian 8, replace "debian10" with debian8
sudo ./asinstall # will install the .deb packages
asadm --version # will show the installed version of asadm
asbackup --version # will show the installed version of asbackup
asrestore --version # will show the installed version of asrestore
aql --version # will show the installed version of aql
```
### Overview
Use this tutorial to install Aerospike Tools on a Debian system using `.deb` packages.

Above is a condensed version, and below are the step-by-step instructions.

{{md (path-resolve (path-dirname page.src) "download.part.md") }}

{{md (path-resolve (path-dirname page.src) "install.part.md") }}

{{md (path-resolve (path-dirname page.src) "validate.part.md") }}
