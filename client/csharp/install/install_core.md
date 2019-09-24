---
title: C# Client Installation for .NET Core
description: Install Aerospike C# Client for .NET Core.
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Install Aerospike C# Client for .NET Core.

### Prerequisites

- NET Core 2.0 and later
- [Optional]Visual Studio 2017 and later

### Nuget Install

The compiled C# client library dll is available on [nuget](https://www.nuget.org/packages/Aerospike.Client). To install, run the following command in the [Package Manager Console](http://docs.nuget.org/docs/start-here/using-the-package-manager-console):

```
PM> Install-Package Aerospike.Client
```

### Source Code Package
 
The C# client source code package is available from:

- [Aerospike Website](/download/client/csharp).  Download package and unzip files.

- [Github](http://github.com/aerospike/aerospike-client-csharp).  Clone source code repository.    

The client projects for .NET Core are located in the `Core` folder.  The contents are:

- **Aerospike.sln** &mdash; Visual Studio solution for .NET Core projects.
- **AerospikeClient** &mdash; C# client library.
- **AerospikeTest** &mdash; C# client unit tests. 

### Build Instructions

	$ cd Core/AerospikeClient
	$ dotnet restore
	$ dotnet build --configuration Release

### Test Instructions

	$ cd Core/AerospikeTest
	$ dotnet restore
	$ dotnet build --configuration Release
	$ dotnet test --configuration Release

### Functionality

This package supports all C# client functionality except:

* Aggregation queries.  Aggregation queries require Lua code to be executed on the client side and NeoLua does not support .NET Core.  The following client methods are not supported:

		public ResultSet QueryAggregate(QueryPolicy policy, Statement statement, string packageName, string functionName, params Value[] functionArgs)
		public void QueryAggregate(QueryPolicy policy, Statement statement, Action<Object> action)
		public ResultSet QueryAggregate(QueryPolicy policy, Statement statement)
	
* Automatic serialization of C# objects to binary.  .NET Core does not support object serialization.  The following bin constructor is not supported:

		public Bin(string name, object value)
	
	Users should serialize these objects to binary using their 3rd party serializer of choice (protobuf, msgpack, etc...) and then use the byte array bin constructor:

		public Bin(string name, byte[] value)
