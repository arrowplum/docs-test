---
title: MacOS Install
description: Install and deploy the Aerospike C client APIs using MacOS.
categories:
  - aerospike-client-c
tags:
  - aerospike-client-c
  - c
---

### Prerequisites

Meet the following prerequisites before installing the Aerospike C client:

- MacOS 10.9+.
- XCode 6+.

If asynchronous functionality is desired, one of the following event frameworks must be installed.

#### [libuv 1.8.0+](http://docs.libuv.org)

libuv has excellent performance and supports all platforms.  The client also supports async
TLS (SSL) sockets when using libuv.

#### [libev 4.24+](http://dist.schmorp.de/libev)

libev has excellent performance on Linux/MacOS, but its Windows implementation
is suboptimal.  Therefore, the C client supports libev on Linux/MacOS only.
The client does support async TLS (SSL) sockets when using libev.

#### [libevent 2.0.22+](http://libevent.org)

libevent is less performant than the other two options, but it does support all
platforms.  The client also supports async TLS (SSL) sockets when using libevent.

If a different event framework version is required, then the C client should be built from 
[source code](https://github.com/aerospike/aerospike-client-c) with that event framework version
instead of following these pre-compiled library installation steps.

### Download And Extract

Download the client package from the [Download]({{book.vars.download-url}}) page.  There are
separate client packages for each event framework. 

| Package | Description |
| ------- | ----------- |
| aerospike-client-c-{VERSION}.mac.x86_64.tgz | sync commands |
| aerospike-client-c-libuv-{VERSION}.mac.x86_64.tgz | sync and async commands with libuv |
| aerospike-client-c-libev-{VERSION}.mac.x86_64.tgz | sync and async commands with libev |
| aerospike-client-c-libevent-{VERSION}.mac.x86_64.tgz | sync and async commands with libevent |

Extract the contents of the chosen package:

```bash
tar xvf <package>
```

Example
```bash
tar xvf aerospike-client-c-libuv-4.3.0.mac.x86_64.tgz
cd aerospike-client-c-libuv-4.3.0.mac.x86_64
```

### Contents

The package contains the following installer packages:

- `aerospike-client-c[-{EVENTLIB}]-devel-{VERSION}.pkg`

  Aerospike C client development (static library and header files) installer for given version and
  event framework library.  This installer is used for developers when compiling/linking their
  applications with the Aerospike client library.

- `aerospike-client-c[-{EVENTLIB}]-{VERSION}.pkg`
  
  Aerospike C client runtime (shared library and lua files) installer for given version and event
  framework library.

### Install

Install one development package to compile your application with Aerospike client.  Install
corresponding runtime package to run your application with Aerospike client.

- Double-click on the installer package in a Finder window.  Example: `aerospike-client-c-libuv-devel-4.3.0.pkg`

A message may appear that the package is from an unidentified developer and can't be opened. 
If that message appears, follow these steps:

  1. Click **OK**. 
  1. Right-click or Ctrl-click the package icon, and select **Open** in the context menu.
  1. Click **Open** in the confirmation dialog box.

Then, follow the installation prompts.

Development files install in:

- `/usr/local/include/aerospike`
- `/usr/local/include/citrusleaf`
- `/usr/local/lib/libaerospike.a`

Runtime files install in:

- `/usr/local/lib/libaerospike.dylib`
- `/usr/local/aerospike/client/sys/udf/lua`
- `/usr/local/aerospike/client/usr/udf/lua`

### Building Applications with Event Library

Event libraries usually install into /usr/local/lib.  Most operating systems do not
search /usr/local/lib by default.  Therefore, the following `LD_LIBRARY_PATH` setting
may be necessary.

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

When compiling your async applications with aerospike header files, the event library
must be defined (`-DAS_USE_LIBUV` or `-DAS_USE_LIBEV` or `-DAS_USE_LIBEVENT`) on the
command line or in an IDE.  Example:

```bash
gcc -DAS_USE_LIBUV -o myapp myapp.c -laerospike -lev -lssl -lcrypto -lpthread -lm -lz
```

### Next Steps
- Run a Read or Write [Example](/docs/client/c/examples)
- Try the [Benchmark Tool](/docs/client/c/benchmarks)
