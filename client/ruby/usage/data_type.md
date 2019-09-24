---
title: Supported Data Types
description: The Aerospike Ruby client supports the string, integer, float, blog, map, lists, and GeoJSON data types.
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
  - ruby
---

The Aerospike database supports these native types:

- String
- Integer
- Float
- Blob
- Map
- List
- GeoJSON

When a value is set in Ruby, the Aerospike Ruby client library automatically determines the best Aerospike native type for storage. All FixedNum numbers fitting 64 bits convert to an internal number format. Strings are stored with UTF-8 encoding.

The Aerospike internal typing system automatically converts data types. Cross-language data type issues are handled internally by the Aerospike Ruby client library. For example, an Aerospike string is stored internally in UTF-8 format. This allows Java and C#, which both use Unicode preferentially, to interact transparently with Ruby (which uses UTF-8).

The Aerospike Ruby client does *not* automatically serialize other data types such as arbitrary object instances. To do this, serialize the object into a byte array and pass it to the library. These blobs must be manually deserialized on retrieval. If these objects are retrieved by another language, they appear as a regular blob. Objects serialized using another languages serialization system appear to Ruby as blobs.

{{#info}} 
We recommend using complex data types based on unchanging Ruby types (such as arrays of integers and strings) or using a higher level serialization system (such as JSON) or a cross-language serializer (such as MessagePack or Google Protocol Buffers).
{{/info}}
