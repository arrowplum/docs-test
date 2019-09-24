---
title: Build on Debian/Ubuntu
description: Use the Aerospike libevent client to write applications to store and retrieve data from an Aerospike database cluster.
breadcrumbs:
  - title: libevent Client
    url: /docs/client/libevent/build
categories:
  - aerospike-client-libevent
tags:
  - aerospike-client-libevent
  - c
  - libevent
---
 
{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent client to write applications to store and retrieve data from an Aerospike database cluster.

### Prerequisites

The Aerospike libevent client library requires the following libraries present on the local machine:

| Library Name | .deb Package | Description |
| --- | --- | --- |
| glibc | glibc | |
| libssl | libssl-dev | Required for RIPEMD160 hash function. |
| libevent | libevent | libevent |

To get the glibc and libssl dependencies:

```bash
sudo apt-get install libc6-dev libssl-dev
```

Getting the libevent requires downloading and building the source.

### Installing libevent

To download and extract the latest 2.0.x libevent library:

```bash
wget "https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz"
tar xvf libevent-2.0.21-stable.tar.gz

```

Make and install the libevent library:

```
cd libevent-2.0.21-stable
./configure PREFIX=/usr
make
sudo make install
```

### Downloading the Aerospike libevent Library

Download the Aerospike libevent client library package from [here]({{book.vars.download-url}}).

The client package for Linux is:

```
citrusleaf_client_libevent2_{VERSION}.tgz
```

To extract the package contents:

```bash
tar xvzf citrusleaf_client_libevent2_{VERSION}.tgz
```

This places the package contents in the following directory:

```
citrusleaf_client_libevent2_{VERSION}
```

### Building

To change to the Aerospike libevent directory and make the library:

```bash
cd citrusleaf_client_libevent2_{VERSION}
make
```

The library is created in the _lib_ directory:

```bash
ls lib
libev2citrusleaf.a

```

### Running Examples

On a successful build, run examples in the _example_ directory:

```bash
cd example4
make
./example4
```

