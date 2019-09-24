---
title: Managing UDFs
description: Leverage the different tools and APIs available for managing Aerospike User-Defined Functions in an Aerospike cluster.
assets: /docs/udf/assets
---

### Overview

Aerospike provides tools such as [aql](/docs/tools/aql/udf_management.html) and
[APIs](/download/client/) for managing User-Defined Functions within a cluster. 

Management of User-Defined Functions is centered around modules.
A modules is a file containing one or many User-Defined Functions.
A module and all it's external (non-Aerospike) dependencies must be uploaded,
and registered with the Aerospike cluster, before the UDF can be invoked.

To run a UDF, you must specify the module name, the name of the function within
the module, and the arguments that will be passed to the function.

### Life-cycle of a UDF

When a package is loaded into the server, it is immediately compiled into
byte-code and made available to subsequent invocations from clients. All client
requests after the update will only use the most recent version of a module.
If a modules was removed, then the module and it's UDFs will no longer be
available.

If a client is in the middle of invoking a UDF when its package is removed or
updated, it will be able to complete the operation without interruption. Once
the function completes however, removed modules can no longer be invoked as it
is no longer available.

A UDF module can be registered into a cluster using tool
[aql](/docs/tools/aql/udf_management.html) as described below. When a UDF module
is registered, it is actually replicated to each node in the cluster, then
registered by each node. The UDF module will be available as each node registers
it.

### Management Options

We provide several options for managing UDFs.
You can use the following tools or client APIs:

* [aql](/docs/tools/aql/udf_management.html) – A command-line utility for
executing commands against an Aerospike cluster with SQL-like commands
* [Language-specific APIs](/download/client/) provide a number of functions that
allow you to programmatically manage User-Defined Functions in a cluster. 

### Module Dependencies

For details on Lua Modules, see [Developing UDF Modules](/docs/udf/developing_lua_modules.html).

A module and its non-Aerospike dependencies must be uploaded to the cluster.
Dependencies can be loaded from your module; Lua makes use of the require()
function to indicate module-dependencies.

Modules should be registered and maintained as part of administrative operations
using a command line tool like [aql](/docs/tools/aql/udf_management.html).

### Invalidating Lua Cache
For performance reasons, Aerospike keeps Lua contexts cached. There are built-in
mechanisms to automatically invalidate caches when newer versions of the UDF
modules are registered. By default, there are 10 such contexts for all
registered UDF's. As the system runs, it will adapt and increase the number of
contexts cached.

In addition, manual mechanism also exist to invalidate Lua context caches
through the use of the asinfo tool:
```
asinfo -v udf-clear-cache:
```
This will invalidate the cache on a single node. For a cluster, this command can be applied to all nodes:
```
asadm -e "asinfo -v 'udf-clear-cache:'"
```

### Operational Notes

UDF modules are stored in the following directory path by default:

`/opt/aerospike/usr/udf/lua`

You can override this via the server configuration, in the mod-lua block:
```
mod-lua {
  user-path /opt/aerospike/usr/udf/lua
}
```
