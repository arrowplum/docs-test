---
title: Windows Install
description: Install and deploy the Aerospike C client APIs using Windows.
categories:
  - aerospike-client-c
tags:
  - aerospike-client-c
  - c
---

### Prerequisites

Meet the following prerequisites before installing the Aerospike C client:

- Windows 7+.
- Visual Studio 2015+.

If asynchronous functionality is desired, one of the following event frameworks should be chosen.
The Windows version of these event libraries are bundled with the C client package and do not
need to be installed separately.

#### [libuv 1.15.0](http://docs.libuv.org)

libuv has excellent performance and supports all platforms.  The client also supports async
TLS (SSL) sockets when using libuv.

#### [libevent 2.1.8](http://libevent.org)

libevent is less performant, but it does support all platforms.  The client also supports async
TLS (SSL) sockets when using libevent.

### Install from NuGet

C client Windows packages can be installed from [NuGet](https://www.nuget.org/packages?q=aerospike-client-c).

- In Visual Studio's Solution Explorer, Right-click on your project and choose "Manage NuGet Packages...".
- Click "Browse" and enter "aerospike-client-c" in your search box.
- Click on one of the following packages.

| Package | Description |
| ------- | ----------- |
| aerospike-client-c-{VERSION} | sync commands |
| aerospike-client-c-libuv-{VERSION} | sync and async commands with libuv |
| aerospike-client-c-libevent-{VERSION} | sync and async commands with libevent |

- Click `Install`
- Click `OK`

Libraries and header files install in:

- `packages/aerospike-client-c[-{EVENTLIB}]-{VERSION}/build/native`

Lua system files install in:

- `packages/aerospike-client-c[-{EVENTLIB}]-{VERSION}/lua`

### Install from Custom Client Build

If installing from a custom C client build (not NuGet install), additional configuration steps are required.

- Define these preprocessor macros in your application project.

| Type | Macros |
| ------- | ----------- |
| sync | AS_SHARED_IMPORT;_CRT_SECURE_NO_DEPRECATE;_TIMESPEC_DEFINED |
| sync and async with libuv | AS_USE_LIBUV;USING_UV_SHARED;AS_SHARED_IMPORT;_CRT_SECURE_NO_DEPRECATE;_TIMESPEC_DEFINED |
| sync and async with libevent | AS_USE_LIBEVENT;EVENT__NEED_DLLIMPORT;AS_SHARED_IMPORT;_CRT_SECURE_NO_DEPRECATE;_TIMESPEC_DEFINED |

- Disable 4996 warning.

- Use aerospike.lib and optional async library (libuv.lib or event_core.lib) and define directory where these libraries exists.

### Next Steps
- Run a Read or Write [Example](/docs/client/c/examples)
- Try the [Benchmark Tool](/docs/client/c/benchmarks)
