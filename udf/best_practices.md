---
title: Lua UDF Best Practices
description: Apply best practices using Aerospike User-Defined Functions with the Aerospike database
assets: /docs/udf/assets
---

### Overview

The following development tips were derived from our experience in developing user-defined functions for Aerospike.

These tips are intended for development environments and should not be employed in production.
Some of these suggestions reduce the performance of the Aerospike server
(extensive logging can reduce the server's performance significantly),
so we recommend you use the system defaults in production environments.

We will assume you have already installed an Aerospike cluster to be used for development. This cluster could be a multi-node environment or single node environment. At Aerospike, many developers use virtual machines (VMs) to form a cluster for development. We recommend a similar scheme, so you can develop in an environment each developer controls. 

A summary of the tips are:

- Disable Caching
- Check for Unintended Variable Definition
- Logging
- Strategic Logging for Debugging
- Error Behavior
- Use Lua Table for Temporary Variables


#### Disable Caching

Aerospike caches UDF files when the they are loaded into the server, and only updates the cache when told to reload the file. For development, we recommend disabling the cache, so each change to the file will be immediately picked up by the server.

The caching can be disabled via Aerospike's configuration file:

```
mod-lua {
  cache-enabled false
}
```

Changes to the server configuration will require a server restart. **Disabling the cache will drastically decrease performance of a
production server**.

#### Check for Unintended Variable Definition

Since Lua is a pure run-time language that does not have any static (compile time) checking, there is no checking for undefined or uninitialized variables.  As a result, any mismatched names (as a result of typos or otherwise) will not be flagged as an error.  Instead, all variables are valid and are nil until they are initialized. To check for such variables, use the following to grep for unintended variables:

```
luac -p -l yourModule.lua  | grep [gs]etGlobal
```

#### Logging

Aerospike has a [log facility](/docs/reference/serverlogmessages/) which allows you to control the verbosity of log messages from various components. The log facility is also used to log messages from UDFs.

The log levels for Aerospike are: DETAIL, DEBUG, INFO, WARNING and CRITICAL. You can set the levels in the [logging block](/docs/reference/configuration/index.html#file) of the configuration file:

```
logging {
  file PATH {
    context any warning
    context udf detail
    context query debug
    context aggr debug
  }
  file /var/log/aerospike/aerospike.log {
    context any info
    context udf critical
    context query critical
    context aggr critical
}
```

Where:

* The PATH is the path to the file to store log messages.
* The **any** context defines the default log level for any undefined context.
* The udf context defines a **detail** log level for UDFs.
* The query context defines a **debug** log level for queries and aggregations.

During development, it is common to use DETAIL or DEBUG levels to show
more detail in the log, and then bypass that additional information
in production mode by switching to the INFO level.
Record UDFs generally require only the **udf** context, but stream UDFs
require both **udf** and **query** to get the full system details.

We use DETAIL as the log level, because log messages emitted from UDF files
will be logged at their corresponding levels.
Both udf and query contexts are required, as udf is for all UDF related logs,
while query is for all Query and Aggregation related logs.

In the Lua, you can utilize the following log functions:

* trace() – log a DETAIL message
* debug() – log a DEBUG message
* info() – log an INFO message
* warn() – log a WARNING message

You can utilize the Lua log functions as:

```
info("Hello %s", "Bob")
debug("There are %d elements in the list", list.size(mylist));
```

When the format string arguments are not strings or integers,
then you need to call tostring() on the variable:

```
local l = list{1,2,3,4}
info("We have %s", tostring(l))
```

The message written to the log file would be like:

	Mar 21 2013 04:36:43 GMT: INFO (query): (/home/bob/lua-devel/test.lua:3) Hello Bob
	Mar 21 2013 04:36:43 GMT: INFO (query): (/home/bob/lua-devel/test.lua:5) We have List(1,2,3,4)

#### Strategic Logging for Debugging

Aerospike currently does not support interactive debugging of Lua code.  So, in the absence of a debugger, the most straight-forward way to see what's going on in the UDF is to exploit the logging mechanism by putting logging statements at key points in the code. A log statement gets expensive when it computes the string of a complex structure that is then fed into the logging function -- and then potentially not used.
We often use **switches** to turn off the expensive logging statements when the UDF is ready for production use. 

#### Error Behavior

If your Lua file has an error, it will also be emitted to the log.

As an example, the following file contains a syntax error at line 3,
due to a common mistake of using {} to specify blocks of code:

```
function foo() {
}
```

The message would be logged as:

```
Oct 31 2013 02:50:54 GMT: WARNING (udf): (base/udf_cask.c:391) udf-put: compile error: [bad.lua:1] unexpected symbol near '{'
```

If your Lua file has a run-time error:

```
function my_record_udf(rec)
  I_dont_exist(rec)
  return 1
end
```

The message will be logged as:

```
Oct 31 2013 03:00:20 GMT: WARNING (udf): (base/udf_rw.c:274) FAILURE when calling mytest my_record_udf ...de/aerospike/modules/mod-lua/src/test/lua/mytest.lua:2: attempt to call global 'I_dont_exist' (a nil value)
```

#### Use Lua Table for Temporary Variables
If variables do not need to be read or stored from Aerospike, which would require data type such as map() or list(), it is best to have a Lua Table object instead.

