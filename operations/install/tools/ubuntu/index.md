---
title: Install on Ubuntu
description: This tutorial covers installing Aerospike Tools on Ubuntu system
---
```bash
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/ubuntu14'
# for ubuntu 16.04, replace "ubuntu14" with ubuntu16
# for ubuntu 18.04, replace "ubuntu14" with ubuntu18
tar -xvf aerospike-tools.tgz
cd aerospike-tools-*-ubuntu14.04
# for ubuntu 16.04, replace "ubuntu14" with ubuntu16
# for ubuntu 18.04, replace "ubuntu14" with ubuntu18
sudo ./asinstall # will install the .deb packages
asadm --version # will show the installed version of asadm
asbackup --version # will show the installed version of asbackup
asrestore --version # will show the installed version of asrestore
aql --version # will show the installed version of aql
```
### Overview
Use this tutorial to install Aerospike Tools on an Ubuntu system using `.deb` packages.

Above is a condensed version, and below are the step-by-step instructions.

{{md (path-resolve (path-dirname page.src) "download.part.md") }}

{{md (path-resolve (path-dirname page.src) "install.part.md") }}

{{md (path-resolve (path-dirname page.src) "validate.part.md") }}


