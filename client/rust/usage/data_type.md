---
title: Supported Data Types
description: The Aerospike database supports several data types including string, integer, blog, map, and lists using the Aerospike Rust client 
categories:
  - aerospike-client-rust
tags:
  - aerospike-client-rust
  - Rust
---

The Aerospike Database supports these native types:

- Integer
- String
- Bytes
- Double
- List
- Map
- GeoJSON

When setting a value in Rust, the Aerospike library automatically determines
the best native Aerospike data type for storage:

- Integers of all types up to and including `i64::MAX` are converted to 64-bit
  numerics.
- `u64` values are not supported as record bin values and need to be casted to
  one of the other supported integer types. `u64` values can be stored as
  elements or keys in lists and maps.
- Floating point values are stored in 64-bit IEEE-754 format.
- Strings are stored as opaque byte arrays but de-serialized as UTF-8 strings
  when reading from the database.
- Byte arrays are stored as blobs.
