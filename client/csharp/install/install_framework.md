---
title: C# Client Installation for .NET Framework
description: Install Aerospike C# Client for .NET Framework on Windows.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Install Aerospike C# Client for .NET Framework on Windows.

### Prerequisites

- Windows 7/Windows Server 2008 and later
- Visual Studio 2010 and later
- .NET 4.0 and later

### Nuget Install

The compiled C# client library dll is available on [nuget](https://www.nuget.org/packages/Aerospike.Client). To install, run the following command in the [Package Manager Console](http://docs.nuget.org/docs/start-here/using-the-package-manager-console):

```
PM> Install-Package Aerospike.Client
```

`AerospikeClient` is downloaded and added to your project as a reference in Visual Studio.  `Neo.Lua` will also be downloaded, but it's not automatically added as a reference.  If you plan on issuing aggregation queries (where lua code is run on the client side), then manually add a reference to `Neo.Lua` located in the downloaded `packages` folder.


### Source Code Package
 
The C# client source code package is available from:

- [Aerospike Website](/download/client/csharp).  Download package and unzip files.

- [Github](http://github.com/aerospike/aerospike-client-csharp).  Clone source code repository.    

The client projects for .NET Framework are located in the `Framework` folder.  The contents are:

- **Aerospike.sln** &mdash; Visual Studio solution for .NET Framework projects.
- **AerospikeClient** &mdash; C# client library.
- **AerospikeDemo** &mdash; C# demonstration program for examples and benchmarks.
- **AerospikeTest** &mdash; C# client unit tests. 
- **AerospikeAdmin** &mdash; Aerospike user administration. This application is only valid for enterprise servers configured to require user authentication.

### Build

The C# client library can optionally be built from source code.  Supported compile targets are AnyCPU, x64 (64-bit), and x86 (32-bit). The solution supports the following configurations:

- Debug
- Release
- Debug IIS : Same as Debug, but store reusable buffers in HttpContext.Current.Items.
- Release IIS : Same as Release, but store reusable buffers in HttpContext.Current.Items.

To build the solution:

1. Double-click **Aerospike.sln**.  The solution opens in Visual Studio.
1. Select **Build > Configuration Manager**.
1. Select the desired solution configuration and platform.
1. Click **Close**.
1. Select **Build > Build Solution**.

After building the solution, the following dlls are in _AerospikeClient/bin/&lt;platform&gt;_.

Name | Description
--- | ---
AerospikeClient.dll | Aerospike C# client library. 
Neo.Lua.dll | Lua 5.3 interpreter. Note: This library is only required when performing queries with aggregate UDFs.
