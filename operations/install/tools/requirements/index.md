---
title: Aerospike Tools Requirements
description: This tutorial covers all requirements for the Aerospike Tools package
---
### Overview
The Aerospike Tools package works with all versions of the Aerospike Server, both Enterprise and Community Editions. 
It runs on 64-bit versions of RHEL, Ubuntu 14.04+, Debian 8+ and Mac OS X.

### Prerequisites
Make sure the following packages are installed:
- openssl
- java 1.8 or greater
- python 2.7+ (< 3)

#### Centos 6
For centos 6, python 2.6 is the default python version. You will need to install python 2.7 and setup the environment to use it. There are different ways to setup python 2.7. Below is one suggested procedure.

##### Install from Software Collection
1. The following are steps to install python 2.7. For further details, refer to [this python 2.7 package release](https://www.softwarecollections.org/en/scls/rhscl/python27/).

    ```bash
    sudo yum update # update yum
    sudo yum install centos-release-scl # install a package with repository
    sudo yum install python27 # install Python 2.7
    ```

2. Once python 2.7 is installed, the environment needs to be set to use it. One of the following two options can be leveraged:

    * Start using the software collections package:
        ```bash
        sudo scl enable python27 bash
        ```

    * Set the following environment variables:
        ```bash
        export LD_LIBRARY_PATH=/opt/rh/python27/root/usr/lib64:$LD_LIBRARY_PATH # update LD_LIBRARY_PATH
        export PATH=/opt/rh/python27/root/usr/bin:$PATH # update PATH
        ```

#### Ubuntu 18
In addition to the above requirements Ubuntu 18 also requires the following package:
- libreadline6

### Python Packages
For tools version 3.15.3.6 and below, the following packages are required to use the advanced features of [`asinfo`](/docs/tools/asinfo/index.html) and [`asadm`](/docs/tools/asadm/index.html).

You can use any python package manager to install the following packages. `pip` is recommended.


| Package | Version | Feature | Use | Description|
|--------|---------|---------|------|------------|
| argparse | - | advanced command line options | - | If not available then the tools will run with the default python option parser.|
| bcrypt | - | Aerospike security | generate password hash | If not available then the tools will not be able to connect to an Aerospike Cluster with security enabled.|
| pexpect | >=3.0 | asadm collectinfo | collect remote system statistics | If not available then `asadm` will collect system statistics for localhost only. Please note that it does not affect Aerospike related data collection.|
| toml | - | Tools configuration file(s) | load tools configuration file(s) | If not available then the tools will ignore all configuration files and continue without them. |
| jsonschema | centos6: 2.5.1, other: >=2.5.1 | Tools configuration file | validate configuration file schema | If not available then the tools will ignore all configuration files and continue without them.|
| pyOpenSSL | >=16.0.0 | SSL Connection | make SSL connection with server | If not available then the tools will not be able to connect to an Aerospike Cluster using SSL.|
| pyasn1 | >=0.3.1 | SSL Connection | validate DNS name for SSL connection | If not available then the tools will not be able to connect to an Aerospike Cluster using SSL.|

{{#note}}These requirements are only for advanced features in the `asinfo` and `asadm` tools. If these are not available then `asinfo` and `asadm` will still be operational outside of some of those more advanced features.{{/note}}
