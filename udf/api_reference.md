---
title: Lua UDF – API Reference
description: Leverage Aerospike APIs for extensions to Lua for functions, modules and types.
assets: /docs/udf/assets
---

### Overview

API Reference for Aerospike extensions to Lua, including functions, modules and types.

#### Types

Aerospike provides a variety of Lua types which coincide with the types supported by the database.

* [bytes](/docs/udf/api/bytes.html) - The bytes type provides the ability
	to build a byte array using bytes and integers. This type coincides with BLOB type in the database.
* [list](/docs/udf/api/list.html) - A list is data structure that represents a sequence of values. 
* [map](/docs/udf/api/map.html) — A collection of (key, value) pairs, in which a key can only appear once in the collection.
* [record](/docs/udf/api/record.html) — Represents database records, including bins – (name, value) pairs – and metadata.
* [stream](/docs/udf/api/stream.html) - Represents streams of records.

#### Modules

The following are modules which provide added functionality.

* [aerospike](/docs/udf/api/aerospike.html) — The aerospike object is a global object that exposes database operations.
* [logging](/docs/udf/api/logging.html) — Logging functions that send log messages to the Aerospike Server's logs. 

