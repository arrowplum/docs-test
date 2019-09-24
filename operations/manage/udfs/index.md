---
title: Managing UDFs
description: Leverage Aerospike tools and APIs for managing user-defined functions in a cluster. 
---


### Overview

Aerospike provides the [aql](/docs/tools/aql/udf_management.html) command line tool as well as APIs (in C, Java, C# .NET, Node.js, PHP, Python, Go, and Ruby) for managing user-defined functions in a cluster.

Management of user-defined functions is centered around modules. A module is a file containing one or more user-defined functions. A module and all its external (non-Aerospike) dependencies must be uploaded and registered with the Aerospike cluster before the UDF can be invoked. A deployment may have one or many modules.

To execute a UDF, you specify the module name, the name of the function within the module, and the arguments that will be passed to the function.

### Lifecycle of a UDF

When a package is loaded into the server, it is immediately compiled into byte-code and made available to subsequent invocations from clients. All client requests after the update will only use the most recently updated module.

If a client is in the middle of invoking a UDF when its module is removed or updated, it will be able to complete the operation without interruption. Once the function completes, any subsequent invocation will either use the updated function, or fail if the module was removed.

A UDF module can be registered into a cluster using the [aql](/docs/tools/aql/) tool. A module can also be registered by using a client API. 

When a UDF module is registered, it is actually replicated to each node in the cluster, then registered by each node. The UDF module will be available when every node registers it. However, since registration is an asynchronous process that occurs within the cluster, there may be a delay between the registration action and the availability of the UDF.  You can check the status of the UDF registration via one of the provided tools. 

UDF registration should be treated as an administrator operation, and be controlled using normal change control procedures. It should not be continuously performed by applications at run-time.

### Management Options

Aerospike provides several options for managing user-defined functions.  You can use the following tools or client APIs:

- [aql](/docs/tools/aql/) – A command-line utility for executing commands against an Aerospike cluster with SQL-like commands
- Language Specific API - provides a number of functions that allow you to programmatically manage user-defined functions in a cluster. This currently includes:
	- [Java Client](/docs/client/java/usage/udf/register.html)
	- [C# Client](/docs/client/csharp/usage/udf/register.html)
	- [C Client](/docs/client/c/usage/udf/register.html)

### Module Dependencies

For details on Lua Modules, see [Lua UDF – Developing Lua Modules](/docs/udf/developing_lua_modules.html).

- A module and its non-Aerospike dependencies must be uploaded to the cluster. Dependencies can be loaded from any module, as Lua makes use of the `require()` function to indicate module dependencies. 
- Modules should be registered and maintained as part of administrative operations using a command line tool like [aql](/docs/tools/aql/).
- Managing UDFs from application code in a high frequency manner should be avoided because the operations are somewhat heavy-weight and can impact system performance.

### UDF Latency

The overall UDF histogram is printed in the log file (`/var/log/aerospike/aerospike.log`) every 10 (by default) seconds.

```asciidoc
Sep 28 2018 14:58:56 GMT: INFO (info): (hist.c:240) histogram dump: {test}-udf (267911 total) msec
Sep 28 2018 14:58:56 GMT: INFO (info): (hist.c:257)  (00: 0000238581)  (01: 0000024013)  (02: 0000003574)  (03: 0000000963) 
Sep 28 2018 14:58:56 GMT: INFO (info): (hist.c:257)  (04: 0000000572)  (05: 0000000174)  (06: 0000000033)  (07: 0000000001)
```

In case the system shows very high latency the following things should be checked:

- Make sure Lua caching is enabled (which is the default behavior -- see [Lua cache config](/docs/reference/configuration/index.html#cache-enabled))
- UDFs generally hold the record lock for a relatively long duration. Check to see if there are hot-keys, i.e. a small set of keys having a lot of UDFs executed on them.

### UDF Statistics

The aql tool can be used to get statistics for a given namespace (e.g. namespace "test")

```sql
aql> stat namespace test
```

UDF related stats:

- **[client_udf_complete](/docs/reference/metrics/#client_udf_complete)**

  Number of successfully completed record UDF transactions.

- **[client_udf_error](/docs/reference/metrics/#client_udf_error)**

  Number of unsuccessful record UDF transactions (other than timeouts).  This includes transactions that fail before executing the UDF.  See the server log file for more information about the error.  Note that the error is also returned to the client.

- **[client_udf_timeout](/docs/reference/metrics/#client_udf_timeout)**

  Number of timed out record UDF transactions.  The timeout error is returned to the client.

- **[client_lang_read_success](/docs/reference/metrics/#client_lang_read_success)**

  Number of successfully completed record UDF read transactions.

- **[client_lang_write_success](/docs/reference/metrics/#client_lang_write_success)**

  Number of successfully completed record UDF write transactions.

- **[client_lang_delete_success](/docs/reference/metrics/#client_lang_delete_success)**

  Number of successfully completed record UDF delete transactions.

- **[client_lang_error](/docs/reference/metrics/#client_lang_error)**

  Number of unsuccessful record UDF transactions that failed during UDF execution.

- **[udf_sub_tsvc_error](/docs/reference/metrics/#udf_sub_tsvc_error)**

  Number of internal UDF sub-transactions (these are for UDF background scans and queries) unsuccessful at the transaction layer (not including transaction queue timeouts).

- **[udf_sub_tsvc_timeout](/docs/reference/metrics/#udf_sub_tsvc_timeout)**

  Number of internal UDF sub-transactions that timed out on the transaction queue.

- **[udf_sub_udf_complete](/docs/reference/metrics/#udf_sub_udf_complete)**

  Number of successfully completed internal UDF sub-transactions.

- **[udf_sub_udf_error](/docs/reference/metrics/#udf_sub_udf_error)**

  Number of internal UDF sub-transactions that failed during processing (other than timeouts).

- **[udf_sub_udf_timeout](/docs/reference/metrics/#udf_sub_udf_timeout)**

  Number of internal UDF sub-transactions that timed out during processing.

- **[udf_sub_lang_read_success](/docs/reference/metrics/#udf_sub_lang_read_success)**

  Number of successfully completed internal UDF read sub-transactions.

- **[udf_sub_lang_write_success](/docs/reference/metrics/#udf_sub_lang_write_success)**

  Number of successfully completed internal UDF write sub-transactions.

- **[udf_sub_lang_delete_success](/docs/reference/metrics/#udf_sub_lang_delete_success)**

  Number of successfully completed internal UDF delete sub-transactions.

- **[udf_sub_lang_error](/docs/reference/metrics/#udf_sub_lang_error)**

  Number of unsuccessful internal UDF sub-transactions that failed during UDF execution.

- **[retransmit_all_udf_dup_res](/docs/reference/metrics/index.html#retransmit_all_udf_dup_res)**

  Number of retransmits that occurred during client initiated UDF transactions that were being duplicate resolved. This includes retransmits originating on the client as well as proxying nodes.

- **[retransmit_all_udf_repl_write](/docs/reference/metrics/index.html#retransmit_all_udf_repl_write)**

  Number of retransmits that occurred during client initiated UDF transactions that were being replica written. This includes retransmits originating on the client as well as proxying nodes.

- **[retransmit_client_udf_dup_res](/docs/reference/metrics/index.html?show-removed=1#retransmit_client_udf_dup_res)**

  Number of retransmits that occurred during client initiated udf transactions that were being duplicate resolved. Replaced with retransmit_all_udf_dup_res as of version 4.5.1.5.

- **[retransmit_client_udf_repl_write](/docs/reference/metrics/index.html?show-removed=1#retransmit_client_udf_repl_write)**

  Number of retransmits that occurred during client initiated udf transactions that were being replica written. Replaced with retransmit_all_udf_repl_write as of version 4.5.1.5.

- **[retransmit_udf_sub_dup_res](/docs/reference/metrics/#retransmit_udf_sub_dup_res)**

  Number of duplicate resolve retransmissions for UDF sub-transactions.

- **[retransmit_udf_sub_repl_write](/docs/reference/metrics/#retransmit_udf_sub_repl_write)**

  Number of replica write retransmissions for UDF sub-transactions.

- **[scan_aggr_complete](/docs/reference/metrics/#scan_aggr_complete)**

  Number of successfully completed scan aggregations.

- **[scan_aggr_error](/docs/reference/metrics/#scan_aggr_error)**

  Number of unsuccessful (non-aborted) scan aggregations.

- **[scan_aggr_abort](/docs/reference/metrics/#scan_aggr_abort)**

  Number of scan aggregations aborted by the user.

- **[scan_udf_bg_complete](/docs/reference/metrics/#scan_udf_bg_complete)**

  Number of successfully completed UDF background scans.

- **[scan_udf_bg_error](/docs/reference/metrics/#scan_udf_bg_error)**

  Number of unsuccessful (non-aborted) UDF background scans.

- **[scan_udf_bg_abort](/docs/reference/metrics/#scan_udf_bg_abort)**

  Number of UDF background scans aborted by the user.

- **[query_udf_bg_success](/docs/reference/metrics/#query_udf_bg_success)**

  Number of successfully completed UDF background queries.

- **[query_udf_bg_failure](/docs/reference/metrics/#query_udf_bg_failure)**

  Number of unsuccessful UDF background queries (including aborts).


Note that all of the above stats are also available at regular intervals via the ticker in the server log file.
The comma-separated values within parentheses for a given UDF stat group are listed in the same order as the descriptions above.

Example log entries:

```asciidoc
Nov 09 2018 00:07:11 GMT: INFO (info): (ticker.c:587) {test} client: tsvc (0,0) proxy (0,0,0) read (140,0,0,3) write (5384,10,0) delete (2417,1,0,1543) udf (35,4,0) lang (26,9,0,4)
Nov 09 2018 00:07:11 GMT: INFO (info): (ticker.c:637) {test} batch-sub: tsvc (0,0) proxy (0,0,0) read (768,0,0,41)
Nov 09 2018 00:07:11 GMT: INFO (info): (ticker.c:664) {test} scan: basic (29,0,0) aggr (0,0,0) udf-bg (7,0,0)
Nov 09 2018 00:07:11 GMT: INFO (info): (ticker.c:688) {test} query: basic (20,1) aggr (6,0) udf-bg (1,0)
Nov 09 2018 00:07:11 GMT: INFO (info): (ticker.c:715) {test} udf-sub: tsvc (0,0) udf (497,90,0) lang (52,304,141,0)
```

### List Registered UDF modules

Using aql

```sql
aql> show modules
```

### Operational Notes

UDF Modules are stored in the following directory path by default:
```
/opt/aerospike/usr/udf/lua
```
You can override this via the server configuration, in the mod-lua block:

```
mod-lua {
  user-path /opt/aerospike/usr/udf/lua
}
```

You will need to ensure this directory is in-sync across the cluster. There are a number of tools you can use to manage this, including configuration management tools.

