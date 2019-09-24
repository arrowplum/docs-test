---
title: Installation
description: Install the Aerospike Python client to build applications to store and retrieve data from an Aerospike cluster.
---

The Aerospike Python Client uses the Aerospike C Client. It is available as a 64-bit Linux variant, and for 64-bit Mac OS 10.9 and above.

Use this guide when installing the Aerospike Python SDK.

## 1- Dependencies

{{#warn}}
If using the latest aerospike python client (>=3.8.0) and pip (>=19.0) on a linux system, only pip is required to install a pre-built 
manylinux2010 wheel.   In this case, [install pip](https://pip.pypa.io/en/stable/installing/) if needed, then skip to the 
[installation section](/docs/client/python/install//index.html#2-installing).
{{/warn}}

- The Aerospike C client. The *pip* program satisfies this dependency by downloading the appropriate version of the client.
- The Python development package.


#### RedHat and CentOS (yum)

The following are dependencies for RedHat Enterprise (RHEL) 6.x, 7.x, CentOS 6.x, 7.x, and related distros using the *yum* package manager.

    sudo yum install python-devel
    sudo yum install openssl-devel
    sudo yum install epel-release
    sudo yum install python-pip

#### Debian and Ubuntu (apt)

The following are dependencies for Ubuntu 14.04+, Debian 6, 7, and related distributions using the *apt* package manager.

    sudo apt-get install python-dev
    sudo apt-get install libssl-dev
    sudo apt-get install python-pip
    sudo apt-get install zlib1g-dev

#### OS X

OS X does not have command line tools. On Mavericks (OS X 10.9) and higher [install command line tools without Xcode](http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/).

    xcode-select --install # install the command line tools, if missing

Install pip

    sudo easy_install pip

Install the dependencies using the OS X [Homebrew](http://brew.sh/) package manager.

    brew install openssl


## 2- Installing 

Use the following code snippet to install the Aerospike Python client:

```bash
pip install aerospike
```

Attempting to install the client with pip for the system default Python may cause permissions issues when copying necessary files. In order to avoid
those issues, the client can be installed for the current user only by specifying "--user" option: 

```bash
pip install --user aerospike
```

For linux installations with aerospike python client version >= 3.8.0, can force an installation from a source build instead of manylinux2010 wheel
download by running the following command:

```bash
pip install aerospike --no-binary :all:
```
