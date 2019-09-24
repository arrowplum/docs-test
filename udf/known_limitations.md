---
title: Known Limitations
description: Lists the known limitations while working with the UDF system
assets: /docs/udf/assets
---

### Lua Limitations
* Please see [Getting Started](/docs/udf/knowing_lua.html) section for Lua modifications.
* Lua version 5.1.4 is used. In 5.1.4 Lua, integers are represented as floats. As a result, NOT all 64-bit integers are representable in Lua. Aerospike plans to upgrade to 5.3 in the future to overcome this short-coming.
* A large number of function parameters (~ >50) causes instability in Lua Runtime Engine.
* The stack buffer limit in Lua is 65500. Thus for example, attempting to pass 20000 4-byte arguments to a Lua function will result in a stack overflow error.
* Integers larger than 2^53 are not guaranteed to be handled accurately because they cannot necessarily be represented in Lua.


### Functional Limitations
* Stream UDF functions which implement map, aggregate, or reduce and Record UDF functions must return one of the types supported by the database: integer, string, bytes, list, and map. It is NOT possible to return a record type.
* A Lua table cannot be used to return key-value pairs.  Instead, the Aerospike defined [map](/docs/udf/api/map.html) type should be used.
* UDFs cannot access or write records with more than 512 bins.
* UDFs within one record scope cannot access another record.
* Stream UDF functions are read-only functions. Only Record UDF functions can be read/write.
* UDF de-registration does not follow the "require" dependency tree. As a result, when deleting "parent.lua", which is required by "child.lua", user must explicitly delete "child.lua" as well.


### Cache Limitations
* 10 Lua states are initially cached on each node for every registered module.  Additional states are cached as needed at runtime up to a maximum of 128 per module.  If 128 or more invocations of the same UDF are running simultaneously on the node, performance will be impacted by any additional invocations of the UDF, as Lua states will be created to service them.
* Cache invalidation happens only when the top-level module is updated. Any update to dependency modules (aka the "required" modules are updated) does not invoke a cache invalidation. Work around is to manually invalidate caches on all nodes. see [Managing UDFs](/docs/udf/managing_udfs.html)
