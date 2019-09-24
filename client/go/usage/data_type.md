---
title: Supported Data Types
description: Aerospike supports the string, integer, blog, map, and list data types.
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
  - go
---

The Aerospike database supports the following native types:

- String
- Integer
- Blob
- Map
- List

When a value is set in Go, the Aerospike library automatically determines the best native type for storage:

- `int` convert to an internal number format. 
- Strings are stored in UTF-8 encoding. 
- `[]byte` is stored as blobs.

The Aerospike internal typing system automatically converts types. Cross-language type issues are handled internally by the Aerospike libraries. For example, an Aerospike string is stored in UTF-8 format to allow Java and C# (which both use Unicode preferentially) to interact transparently with Go, Python, and Ruby (which use UTF-8) and C (which does not have a standard internal character encoding).

{{#note}}
To avoid all format conversion, Aerospike stores the Go `[]byte` type as a pure blob, and presents it to all other languages as a collection of bytes.
{{/note}}

The Aerospike Go client does _not_ automatically serialize any other data types (for example, arbitrary structs). To do this, the application must serialize the object into a byte array and pass it to the library.

Blobs must be manually deserialized on retrieval. If another language retrieves these blobs, they appear as a regular blob. Objects serialized using another language serialization system appear to Go as a blob.

{{#info}} We recommend using complex data types based on unchanging Go types (such as arrays of integers and strings) or a high-level serialization system (such as Gob), or a cross-language serializer like MessagePack or Google Protocol Buffers).{{/info}}
