---
title: C# Client | API
description: Use the Aerospike C# Client API with the Aerospike database.
breadcrumbs:
  - title: Home
    url: /
  - title: Documentation
    url: /docs
  - title: C# Client
    url: /docs/client/csharp
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

The C# client library source code is located in the AerospikeClient project. The library supports the following:

- Aerospike two servers.
  - Synchronous mode using `AerospikeClient` (_Main/AerospikeClient.cs_)
  - Asynchronous mode using `AsyncClient` (_Async/AsyncClient.cs_)
  - Connection pools for each server node
  - Key/Value
  - Batch Key/Value
  - Scan of namespace/set
- Aerospike three servers. This solution includes Aerospike 2 functionality plus:
  - [UDFs](abbr:user-defined functions)
	UDF code is written in the Lua programming language.
  - Secondary index queries
  - Secondary index queries with UDF aggregations
  - Secondary index query/UDF execute

### API Reference

The API reference documentation is located online at: 

https://www.aerospike.com/apidocs/csharp/Index.html

This documentation is included in the client project:

```
Help/Index.html
```

### Supported Data Types

The Aerospike database supports the following native types:

- String
- Integer
- Blob
- Map
- List

The Aerospike C# client library automatically determines the best Aerospike native data type for storage:
- Integers and longs convert to an internal number format
- Strings convert to UTF-8. 
- Byte arrays are stored as blobs.

The benefit of Aerospike’s internal typing system is that data types automatically convert, and cross-language data type issues are handled internally. For example, an Aerospike string is internally stored in UTF-8 format. This allows C# and Java &ndash; which both use Unicode preferentially &ndash; to interact transparently with Python and Ruby (which use UTF-8), and C (which does not have standard internal character encoding).

To avoid all data type conversions, the `byte[]` type is stored as a pure blob, and is presented to all other languages as a collection of bytes.

The Aerospike C# client passes any other type through the C# serialization system, which reduces them to C# blobs. These blobs are then automatically deserialized on retrieval. If they are retrieved by another language, they appear as a blob. In the same way, objects serialized through other serialization systems appear as a blob.

{{#note}}
Use care with C# serialization. Unless you override object signatures, it is very easy to store complex types which cannot be deserialized when on applicaton update. If you serialize an object and store it on the Aerospike server and then change the object class, there is a version mismatch on data reads. If you do a lot of serialization, then serialize the data yourself and pass it through Aerospike as a `byte[]`.

We recommend using complex types based on unchanging C# types (such as arrays of integers and strings), or using a higher level serialization system (such as a cross-language serializer like MessagePack).{{/note}}
