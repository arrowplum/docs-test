---
title: Supported Data Types
description: The Aerospike database supports several data types including string, integer, blog, map, and lists using the Aerospike Java client
categories:
  - aerospike-client-java
tags:
  - aerospike-client-java
  - java
---

The Aerospike Database supports these native types:

- String
- Integer
- Blob
- Map
- List

When setting a value in Java, the Aerospike library automatically determines the best native Aerospike data type for storage:

- Integers and longs are converted to an internal number format.
- Strings are converted to UTF-8.
- Byte arrays are stored as blobs.

Aerospike’s internal typing system automatically converts data types. Cross-language data type issues are handled internally by the Aerospike libraries. For example, an Aerospike String is stored internally in  UTF-8 format, allowing Java and C#&ndash;which both use Unicode preferentially&ndash;applications to transparently interact with Python and Ruby, which use UTF-8, and C, which does not use standard internal character encoding.

{{#note}}
To avoid format conversion, the Java `byte[]` type is stored as a pure blob and presented to all other languages as a collection of bytes.
{{/note}}

Aerospike passes any other type (such as complex Java collection types) through the Java serialization system, and reduces them to Java blobs. On retrieval, blobs are automatically deserialized. If retrieved by another language, they appear as a blob. Objects serialized through other serialization systems appear as a blob to Java.

{{#info}} 
Be careful using Java serialization. Unless you override object signatures, it is very easy to store complex data types that cannot be deserialized on application updates. If you serialize an object and store it on the Aerospike server and then change the object class, on data reads there will be a version mismatch. This is common to any Java serialization. If you do a lot of serialization, then you might want to serialize the data yourself and pass it through Aerospike as a blob. 

We recommend using complex data types based on unchanging Java data types such as arrays of integers and strings, or using a higher level serialization system such as a cross-language serializer (for example, [MessagePack](http://msgpack.org) or [Google’s Protocol Buffers](https://developers.google.com/protocol-buffers).
{{/info}}
