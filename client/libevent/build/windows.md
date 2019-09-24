---
title: Build on Windows
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

### Prerequisite

The Aerospike libevent client library for Windows requires the following libraries present on the local machine:

| Library Name | Description |
| --- | --- | 
| openSSL |  Required for RIPEMD160 hash function. |
| pthread |  Required for pthread mutex. |
| libevent | libevent |

### Installing libevent

To download and extract the latest 2.0.x libevent library:

```bash
wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
tar xvf libevent-2.0.21-stable.tar.gz

```

To build from the command line:

```bash
cd libevent-2.0.X-stable
nmake -f makefile.nmake
```

To build from Visual C++:

1. Select **File>New>Project**.
1. On a general node, pick the Makefile Project template.
 - Name = libevent-2.0.X-stable, 
 - Location = parent directory (libevent-2.0.X-stable). 
1. Click **OK**. 
1. Click **Next**. 
1. Type the build command at the command prompt
```
> nmake -f makefile.nmake 
```
1. Build the project.

### Installing OpenSSL

1. Download OpenSSL from [here](https://www.openssl.org/related/binaries.html).
1. Double-click the exe and follow the installation instructions.

### Installing pthread
 
Download pthread, including the _lib_ and _dll_ folders from [here](ftp://sourceware.org/pub/pthreads-win32/dll-latest)

### Downloading the Aerospike libevent Library

Download the client package from [here]({{book.vars.download-url}}).

The client package for Windows is:

```
citrusleaf_client_winlibevent2_{VERSION}.zip
```

Extract the package contents:

```bash
unzip citrusleaf_client_winlibevent2_{VERSION}.zip
```

This places the package contents in the directory:

```
citrusleaf_client_winlibevent2_{VERSION}.zip
```

### Building

1. Open the visual studio solution file _cl\_libevent2\window_ which contains these two projects:
  - CitrusleafLibrary &mdash; The Aerospike libevent client library files.
  - CitrusleafDemo &mdash; The demo load application built using the Aerospike libevent client library.
1. Start the solution and add all dependencies:
  **CitrusleafLibrary**
   - Header files &mdash; Edit and add `CitrusleafLibrary -> Properties -> Configuration Properties -> C/C++ -> General -> Additional Include Directories`
   - `{path of libevent2_folder}\include`
   - `{path of libevent2_folder}\WIN32-Code`
   - `{path of openSSL_folder}\include`
   - `{path of pthread_folder}\include`
  - Libraries &mdash; Edit and add `CitrusleafLibrary -> Properties -> Configuration Properties -> Librarian -> Additional Library Directories`
   - `{path of libevent2_folder}`
   - `{path of pthread_folder}`
  **CitrusleafDemo**
   - Header files &mdash; Edit and add `CitrusleafDemo -> Properties -> Configuration Properties -> C/C++ -> General -> Additional Include Directories`
   - `{path of libevent2_folder}\include`
  - Libraries &mdash; Edit and add `CitrusleafDemo -> Properties -> Configuration Properties -> Linker -> Additional Library Directories`
   - `{path of libevent2_folder}`
   - `{path of pthread}\lib\x64`
   - `{path of openSSL}\lib`
1. Build the solution. 

### Running Examples

Once the `CitrusleafDemo` project builds successfully:
 
1. In the _CitrusleafDemo.cpp_ file, set the config parameters (such as namespace, set, hostname, and so on).
2. Compile the _CitrusleafDemo_ project and run.

