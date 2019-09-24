---
title: Incompatible API Changes
description: Compatibility issues with the Aerospike C client.
---

### **Version 4.6.0 - Read consistency level changes**

**Read policy changes for transactions involving AP (availability) namespaces**

- Policy variable `as_policy_consistency_level consistency_level` moved to `as_policy_read_mode_ap read_mode_ap`.
- Enum value `AS_POLICY_CONSISTENCY_LEVEL_ONE` moved to `AS_POLICY_READ_MODE_AP_ONE`.
- Enum value `AS_POLICY_CONSISTENCY_LEVEL_ALL` moved to `AS_POLICY_READ_MODE_AP_ALL`.

**Read policy changes for transactions involving SC (strong consistency) namespaces**

- Policy variable `bool linearize_read` moved to `as_policy_read_mode_sc read_mode_sc`.
- Add `as_policy_read_mode_sc` enum below:

```cpp
/**
 * Read policy for SC (strong consistency) namespaces.
 *
 * Determines SC read consistency options.
 */
typedef enum as_policy_read_mode_sc_e {

	/**
	 * Ensures this client will only see an increasing sequence of record versions.
	 * Server only reads from master.  This is the default.
	 */
	AS_POLICY_READ_MODE_SC_SESSION,

	/**
	 * Ensures ALL clients will only see an increasing sequence of record versions.
	 * Server only reads from master.
	 */
	AS_POLICY_READ_MODE_SC_LINEARIZE,

	/**
	 * Server may read from master or any full (non-migrating) replica.
	 * Increasing sequence of record versions is not guaranteed.
	 */
	AS_POLICY_READ_MODE_SC_ALLOW_REPLICA,

	/**
	 * Server may read from master or any full (non-migrating) replica or from unavailable
	 * partitions.  Increasing sequence of record versions is not guaranteed.
	 */
	AS_POLICY_READ_MODE_SC_ALLOW_UNAVAILABLE,

} as_policy_read_mode_sc;
```


### **Version 4.5.0 - Remove aerospike_has_double()**

`aerospike_has_double()` has been removed because all servers that this client supports (servers >= 3.6.0), also support the double data type.


### **Version 4.3.20 - Remove as_policy_batch.use_batch_direct**

`as_policy_batch.use_batch_direct` was removed because the server has removed support for the old batch direct protocol.  Remove any references to this policy field.


### **Version 4.3.16 - Remove lua system_path config**

`as_config_lua.system_path` was removed because lua system code is now loaded directly from C strings instead of files.  Remove any references to lua system_path in your code since those references are no longer necessary.


### **Version 4.2.0 - LDT Removal and Policy Changes**

All large data type (LDT) client functions have been removed as of server version 3.15.

The **[policy definitions](https://github.com/aerospike/aerospike-client-c/blob/master/src/include/aerospike/as_policy.h)** have been redefined to support greater control on timeout behavior and provide defaults that improve retry success.

**New generic `as_policy_base`**

`as_policy_base` is now included in all applicable transaction policies (`as_policy_read`, `as_policy_write`, `as_policy_scan` ...).

```cpp
typedef struct as_policy_base_s {

	/**
	 * Socket idle timeout in milliseconds when processing a database command.
	 *
	 * If socket_timeout is not zero and the socket has been idle for at least socket_timeout,
	 * both max_retries and total_timeout are checked.  If max_retries and total_timeout are not
	 * exceeded, the transaction is retried.
	 *
	 * If both socket_timeout and total_timeout are non-zero and socket_timeout > total_timeout,
	 * then socket_timeout will be set to total_timeout.  If socket_timeout is zero, there will be
	 * no socket idle limit.
	 *
	 * Default: 0 (no socket idle time limit).
	 */
	uint32_t socket_timeout;

	/**
	 * Total transaction timeout in milliseconds.
	 *
	 * The total_timeout is tracked on the client and sent to the server along with
	 * the transaction in the wire protocol.  The client will most likely timeout
	 * first, but the server also has the capability to timeout the transaction.
	 *
	 * If total_timeout is not zero and total_timeout is reached before the transaction
	 * completes, the transaction will return error AEROSPIKE_ERR_TIMEOUT.
	 * If totalTimeout is zero, there will be no total time limit.
	 *
	 * Default: 1000
	 */
	uint32_t total_timeout;

	/**
	 * Maximum number of retries before aborting the current transaction.
	 * The initial attempt is not counted as a retry.
	 *
	 * If max_retries is exceeded, the transaction will return error AEROSPIKE_ERR_TIMEOUT.
	 *
	 * WARNING: Database writes that are not idempotent (such as "add")
	 * should not be retried because the write operation may be performed
	 * multiple times if the client timed out previous transaction attempts.
	 * It's important to use a distinct write policy for non-idempotent
	 * writes which sets max_retries = 0;
	 *
	 * Default: 2 (initial attempt + 2 retries = 3 attempts)
	 */
	uint32_t max_retries;

	/**
	 * Milliseconds to sleep between retries.  Enter zero to skip sleep.
	 * This field is ignored in async mode.
	 *
	 * Reads do not have to sleep when a node goes down because the cluster
	 * does not shut out reads during cluster reformation.  The default for
	 * reads is zero.
	 *
	 * Writes need to wait for the cluster to reform when a node goes down.
	 * Immediate write retries on node failure have been shown to consistently
	 * result in errors. The default for writes is 500ms.
	 */
	uint32_t sleep_between_retries;

} as_policy_base;
```

**Transaction Policy Changes**

- `socket_timeout` (when used) moved to `as_policy_base.socket_timeout`
- `timeout` moved to `as_policy_base.total_timeout`
- `retry` moved to `as_policy_base.max_retries`.  Also default is now 2.
- `sleep_between_retries` moved to `as_policy_base.sleep_between_retries`
- `retry_on_timeout` deleted.  This functionality can be simulated by setting `total_timeout` to zero.
- `replica` now defaults to AS_POLICY_REPLICA_SEQUENCE.

**Global Policy Changes**

Global policy field defaults (like `timeout`) have been deleted from `as_policies`.  If a global `timeout` is needed for all policies, then the timeout must be defined for each default transaction policy.

**Example Policy Conversion**

**Old**

```cpp
	as_config config;
	as_config_init(&config);

    // Set global defaults for all policies.
	config.policies.timeout = 100;
	config.policies.retry = 3;
	config.policies.sleep_between_retries = 10;

	// Set transaction policy defaults which override global defaults.
	config.policies.read.timeout = 50;
	config.policies.read.consistency_level = AS_POLICY_CONSISTENCY_LEVEL_ALL;
	config.policies.write.timeout = 150;
	config.policies.scan.timeout = 0;
	config.policies.query.timeout = 0;

	aerospike_init(client, &config);
```

**New**
```cpp
	as_config config;
	as_config_init(&config);

	// Set transaction policy defaults.
	config.policies.read.base.total_timeout = 50;
	config.policies.read.base.max_retries = 3;
	config.policies.read.base.sleep_between_retries = 10;
	config.policies.read.consistency_level = AS_POLICY_CONSISTENCY_LEVEL_ALL;

	config.policies.write.base.total_timeout = 150;
	config.policies.write.base.max_retries = 3;
	config.policies.write.base.sleep_between_retries = 10;

	config.policies.operate.base.total_timeout = 100;
	config.policies.operate.base.max_retries = 3;
	config.policies.operate.base.sleep_between_retries = 10;

	config.policies.remove.base.total_timeout = 100;
	config.policies.remove.base.max_retries = 3;
	config.policies.remove.base.sleep_between_retries = 10;

	config.policies.apply.base.total_timeout = 100;
	config.policies.apply.base.max_retries = 3;
	config.policies.apply.base.sleep_between_retries = 10;

	config.policies.query.base.socket_timeout = 30000;
	config.policies.query.base.total_timeout = 0;

	config.policies.scan.base.socket_timeout = 30000;
	config.policies.scan.base.total_timeout = 0;

	aerospike_init(client, &config);
```


### **Version 4.0.3 - Max connections is a now a hard limit**

Past versions used to create new connections when a node's outstanding connections exceeded `as_config.max_threads`.  This connection would then be closed (not put back into connection pool) when the transaction finished.  This could handle unexpected bursts of transactions (with a performance penalty), but it also resulted in excessive socket opens/closes and too many sockets in a CLOSED/WAIT state.

Version 4.0.3 renames `as_config.max_threads` to `as_config.max_conns_per_node` and enforces a strict limit on maximum number of connections allowed per server node.  Synchronous transactions will now fail with `AEROSPIKE_ERR_NO_MORE_CONNECTIONS` (after going through retry logic) if the maximum number of connections would be exceeded.

The number of connections used per node depends on how many concurrent threads issue database commands plus sub-threads used for parallel multi-node commands (batch, scan, and query). One connection will be used for each thread.

### **Version 4.0.1 - Asynchronous pipeline callback**

Asynchronous commands can use an exclusive connection from a connection pool or share a pipeline connection from a pipeline connection pool.  In the past, asynchronous methods included a "bool pipeline" argument.  This argument has been changed to a function callback.

The extra pipeline callback allows the user to add more commands to be pipelined directly after the socket write from the calling command has completed.  Asynchronous commands that don't need pipelining should pass in NULL for the pipeline callback.

**Old**
```cpp
void put_callback(as_error* err, void* udata, as_event_loop* event_loop)
{
	// Put command has completed (socket write, socket read and parse result or error).
}

aerospike_key_put_async(as, &err, NULL, &key, &rec, put_callback, NULL, event_loop, true);
```

**New**
```cpp
void put_callback(as_error* err, void* udata, as_event_loop* event_loop)
{
	// Put command has completed (socket write, socket read and parse result or error).
}

void pipeline_callback(void* udata, as_event_loop* event_loop)
{
	// Socket write of put command completed.  Optionally issue more commands to be pipelined. 
}

aerospike_key_put_async(as, &err, NULL, &key, &rec, put_callback, NULL, event_loop, pipeline_callback);
```

### **Version 3.0.96 - Query rename**

An `as` prefix was added to query predicate macros to prevent conflicts with other code:

**Old**

```cpp
    as_query_where_init(&query, 3);
    as_query_where(&query, "bin1", string_equals("abc"));
    as_query_where(&query, "bin1", integer_equals(123));
    as_query_where(&query, "bin1", integer_range(0,123));
```

**New**

```cpp
    as_query_where_init(&query, 3);
    as_query_where(&query, "bin1", as_string_equals("abc"));
    as_query_where(&query, "bin1", as_integer_equals(123));
    as_query_where(&query, "bin1", as_integer_range(0,123));
```

An `as_error` argument was added to `admin` commands, so an error message is now available.  Also, `as_user_roles` was renamed to `as_user`.

**Old**

```cpp
	as_user_roles* user;
	int status = aerospike_query_user(as, policy, user_name, &user);

	if (status == 0) {
		// Process user.
		as_user_roles_destroy(user);
	}
	else {
		printf("Error: %d\n", status);
	}
```

**New**

```cpp
	as_er ror err;
	as_user* user;
	as_status status = aerospike_query_user(as, &err, policy, user_name, &user);

	if (status == AEROSPIKE_OK) {
		// Process user.
		as_user_destroy(user);
	}
	else {
		printf("Error: %d : %s\n", status, err.message);
	}
```


### **Version 3.0.86 - Client policy resolution**

- The Aerospike C client policy resolve at the transaction level is now very easy. If the passed-in policy is not NULL, use it. If the passed-in policy is NULL, use the default policy.

```cpp
if (! policy) {
    policy = &as->config.policies.write;
}
```

- Transaction policy resolve used to involve validating each field and setting undefined fields to the operation policy (`as->config.policies.write.timeout`) defaults or global defaults (`as->config.timeout`) when the operation policy fields were also undefined.  

- The client now resolves global policies once, at startup.

- The policy `UNDEF` enum values were deleted as they are no longer used. 

- The `as_policy_bool` enum was deleted and replaced with `type bool`.

- The policy init functions (`as_policy_write_init()`) now initialize values to actual defaults, not undefined.

- Policy copy functions (`as_policy_write_copy()`) were added to enable creating modified policies based on defaults:

```cpp
as_policy_write policy;
as_policy_write_copy(&as->config.policies.write, &policy);
policy.gen = AS_POLICY_GEN_EQ;
```


### **Version 3.0.86 - Client logging API change**

The client Logging API changed as follows:

**Old**

```cpp
as_log_set_level(&as->log, AS_LOG_LEVEL_INFO);
as_log_set_callback(&as->log, as_client_log_callback);

cf_set_log_level(CF_INFO);
cf_set_log_callback(cf_client_log_callback);
```

**New**

```cpp
as_log_set_level(AS_LOG_LEVEL_INFO);
as_log_set_callback(as_client_log_callback);
```
