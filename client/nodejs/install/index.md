---
title: Installation
description: Install the Aerospike  Node.js client.
---

The Aerospike Node.js client uses the Aerospike C client, and is available as a
64-bit Linux variant and for 64-bit Mac OS 10.8 and later. The Aerospike
Node.js client add-on module is written using V8. To compile V8, you must have
g++ installed in the system.

The Aerospike Node.js client is dependent on the Aerospike C client, which is
downloaded during the installation. To download the Aerospike C client, curl is
required (see [Aerospike C Client Installation](/docs/client/c/install)).

### Prerequisites

The client is compatible with Node.js v4.x (LTS), v6.x (LTS), v8.x (LTS) and
v10.x. It supports the following operating systems: CentOS/RHEL 6/7, Debian
7/8, Ubuntu 12.04/14.04/16.04, as well as many Linux destributions compatible
with one of these OS releases. macOS is also supported.

#### Libraries

The client library requires the following libraries to be present on the machine for building and running:

|Library Name |	.rpm Package | Description
|--|--|--
| libssl | openssl |
|libcrypto | openssl | Required for the `RIPEMD160` hash function.
|liblua5.1 | lua | Required for Lua execution, used in query aggregation.

{{#note}}
Lua is used for query aggregation. If the application is not using aggregation, you can skip Lua installation.
{{/note}}

##### CentOS/RHEL 6.x

To install library prerequisites using `yum`:

```bash
sudo yum install openssl-devel lua-devel
```
Some CentOS installation paths do not include required C development tools. These packages may also be required:

```bash
sudo yum install gcc gcc-c++
```

##### Debian 6+

To install library prerequisites using `apt-get`:

```bash
sudo apt-get install libssl0.9.8 libssl-dev liblua5.1-dev
```

Create the following symlinks to compile the Aerospike Node.js client examples:

```bash
sudo ln -s /usr/lib/liblua5.1.so /usr/lib/liblua.so
sudo ln -s /usr/lib/liblua5.1.a /usr/lib/liblua.a
```

##### Ubuntu 12.04+

To install library prerequisites using `apt-get`:

```bash
sudo apt-get install libssl0.9.8 libssl-dev liblua5.1-dev
```

Create the following symlinks to compile the Aerospike Node.js client examples:

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/liblua5.1.so /usr/lib/liblua.so
sudo ln -s /usr/lib/x86_64-linux-gnu/liblua5.1.a /usr/lib/liblua.a
```

##### Mac OS X

Ensure that your system meets hese prerequisites:

- Mac OS X 10.8 or later
- Xcode 5 or later
- Lua 5.1.5 library (when using query aggregation).

**Openssl Library**

```bash
$ brew install openssl
$ brew link openssl --force
```

**Lua Installation**

Lua is required for query result aggregation.

This example installs Lua 5.1:

```bash
$ curl -O http://www.lua.org/ftp/lua-5.1.5.tar.gz
$ tar xvf lua-5.1.5.tar.gz
$ cd lua-5.1.5
$ make macosx
$ make test
$ sudo make install
```



### Installation

The Aerospike Node.js client is an add-on module that uses the Aerospike C client. Installation first builds the add-on module. The build resolves the Aerospike C client dependency.

You can install the Aerospike Node.js client like any other Node.js module.

### npm Registry Installations

To install `aerospike` as a dependency of your project, in your project directory run:

```bash
$ npm install aerospike
```

To add `aerospike` as a dependency in _package.json_, run:

```bash
$ npm install aerospike --save-dev
```

To require the module in your application:
```bash
const Aerospike = require('aerospike')
```

### Git Repository Installations

When using cloned a repository, install `aerospike` it as a dependency of your application. Instead of referencing the module by name, you reference it by path.

{{#note}}
Install the Aerospike Node.js module locally.
{{/note}}

To install the module as a dependency of your application, run the following in the application directory:

```bash
$ npm install <PATH>
```

where
- `<PATH>` is the path to the Aerospike Node.js client Git respository.

To require it in the application:

```bash
const Aerospike = require('aerospike')
```

### C Client Resolution

When running `npm install`, `npm link` or `node-gyp rebuild`, the *.gyp* script runs `scripts/aerospike-client-c.sh` to resolve the C client dependency. The script checks that the following files are in the search paths:

- `lib/libaerospike.a`
- `include/aerospike/aerospike.h`

The default search paths are:

- `./aerospike-client-c`
- `/usr`

If neither search paths are found, the C client automatically downloads and creates the `./aerospike-client-c` directory.

You can modify C client resolution by forcing a C client download or specifying a custom search path.

#### Forcing a C Client Download

To force a C client download using the `DOWNLOAD=1` environment variable:

```bash
$ DOWNLOAD=1 npm install
```

#### Custom Search Paths

If you have the Aerospike C client installed in a non-standard location or built from source, use the `PREFIX=<PATH>` environment variable to specify the search path for use during the build step.

{{#note}}
`<PATH>` must be the path to a directory containing lib and must include subdirectories.
{{/note}}

To specify the path to the Aerospike C client build directory:

```bash
$ export PREFIX=~/aerospike-client-c/target/Linux-x86_64
```
