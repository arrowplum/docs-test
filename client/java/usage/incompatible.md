---
title: Incompatible API Changes
description: Compatibility issues with the Aerospike Java client.
---

####**Version 4.4.0 - Read consistency level changes**

**Read policy changes for transactions involving AP (availability) namespaces**

- Policy variable `ConsistencyLevel consistencyLevel` moved to `ReadModeAP readModeAP`.

**Read policy changes for transactions involving SC (strong consistency) namespaces**

- Policy variable `boolean linearizeRead` moved to `ReadModeSC readModeSC`.
- Add `ReadModeSC` enum below:

```java
/**
 * Read policy for SC (strong consistency) namespaces.
 * <p>
 * Determines SC read consistency options.
 */
public enum ReadModeSC {
    /**
     * Ensures this client will only see an increasing sequence of record versions.
     * Server only reads from master.  This is the default.
     */
    SESSION,

    /**
     * Ensures ALL clients will only see an increasing sequence of record versions.
     * Server only reads from master.
     */
    LINEARIZE,

    /**
     * Server may read from master or any full (non-migrating) replica.
     * Increasing sequence of record versions is not guaranteed.
     */
    ALLOW_REPLICA,

    /**
     * Server may read from master or any full (non-migrating) replica or from unavailable
     * partitions.  Increasing sequence of record versions is not guaranteed.
     */
    ALLOW_UNAVAILABLE
}
```


####**Version 4.3.0 - Remove obsolete code**

The following old obsoleted code has been removed.
 
- `AsyncClient`: This old async class was replaced by the new async methods in AerospikeClient.  Users should migrate to the new async functionality.

- `Statement.setFilters()` and `Statement.getFilters()`:  These methods were replaced by `Statement.setFilter()` and `Statement.getFilter()`.  The server accepts only one filter.

- `Value.UseDoubleType`: This configuration variable is no longer necessary because all server versions that this client supports also support the double type.  This client supports server versions >= 3.6.0.

- `ClientPolicy.requestProleReplicas`:   This configuration variable is no longer necessary because prole replicas are now always requested.

- Batch direct protocol code: This old batch direct protocol code was replaced by the batch index functionality.  This code was no longer accessible by the external API.

####**Version 4.2.3 - Remove BatchPolicy.useBatchDirect**

`BatchPolicy.useBatchDirect` was removed because the server has removed support for the old batch direct protocol.  Remove any references to this policy field.

####**Version 4.2.0 - Require minimum java version 1.8**

We require minimum java version 1.8.

####**Version 4.1.0 - LDT Removal and Index Exceptions**

LDT methods have been removed on client because large data types are no longer supported by Aerospike Server.

Index exception handling has changed on the client.  In the past, the client swallowed INDEX_ALREADY_EXISTS errors on createIndex().  The client now throws an AerospikeException when an index already exists on createIndex().

The client also used to swallow INDEX_NOTFOUND errors on dropIndex().  The client now throws an AerospikeException when an index does not already exist on dropIndex().

To preserve old behavior, calling code can be changed to:

```java
try {
    IndexTask task = client.createIndex(policy, ns, set, indexName, binName, indexType);
    task.waitTillComplete();
}
catch (AerospikeException ae) {
    if (ae.getResultCode() != ResultCode.INDEX_ALREADY_EXISTS) {
        throw ae;
    }
}

try {
	client.dropIndex(policy, ns, set, indexName);		
}
catch (AerospikeException ae) {
    if (ae.getResultCode() != ResultCode.INDEX_NOTFOUND) {
        throw ae;
    }
}
```

####**Version 4.0.0 - Policy changes**

The **[Policy](https://github.com/aerospike/aerospike-client-java/blob/master/client/src/com/aerospike/client/policy/Policy.java)** class has been redefined to support greater control on timeout behavior and provide defaults that improve retry success.

**Split `timeout` into `socketTimeout` and `totalTimeout`**

socketTimeout is the socket idle timeout in milliseconds.  For synchronous methods, socketTimeout is the socket timeout (SO_TIMEOUT).  For asynchronous methods, the socketTimeout is implemented using a HashedWheelTimer and socketTimeout is only used if totalTimeout is not defined.

If socketTimeout is not zero and the socket has been idle for at least socketTimeout, both maxRetries and totalTimeout are checked.  If maxRetries is not exceeded and totalTimeout is not reached, the transaction is retried.  If both socketTimeout and totalTimeout are non-zero and socketTimeout > totalTimeout, then socketTimeout will be set to totalTimeout.  If socketTimeout is zero, there will be no socket idle limit.

totalTimeout is the maximum time allowed in milliseconds to process a transaction.  totalTimeout is tracked on the client and sent to the server along with the transaction in the wire protocol.  The client will most likely timeout first, but the server also has the capability to timeout the transaction.  If totalTimeout is zero, there will be no total time limit.

To keep old timeout behavior, set socketTimeout equal to totalTimeout.

**Removed `retryOnTimeout`**

retryOnTimeout has been removed because it is no longer necessary.  If a transaction timed out on socketTimeout, retries will now occur until maxRetries is exceeded or totalTimeout is reached.

**Changed Policy `replica` default to `Replica.SEQUENCE`**

The sequence replica algorithm starts with the node containing the key's master partition.  If a connection error occurs, the node containing the key's replicated partition is used on retry.  Writes now switch to replicated nodes on connection errors because the replicated node is most likely to become the new master when the old master goes down.

Reads also switch to replicated nodes on timeouts, but writes do not switch nodes on timeouts.  Timeouts are not a good indicator of downed nodes.

**Changed Policy `maxRetries` default to 2**

maxRetries now defaults to 2 to give a better chance of retry success.

**Changed Policy `sleepBetweenRetries` default to zero for reads**

Reads do not have to sleep when a node goes down because the cluster does not shut out reads during cluster reformation.

The sleepBetweenRetries default for writes remains at 500ms.  Writes need to wait for the cluster to reform when a node goes down.  Immediate write retries on node failure have been shown to consistently result in errors.

**Changed ClientPolicy `requestProleReplicas` default to true.**

Prole replica maps are required when using `Replica.SEQUENCE` algorithm.


####**Version 3.2.0 - Max connections is a now a hard limit**

Past versions used to create new connections when a node's outstanding connections exceeded `ClientPolicy.maxThreads`.  This connection would then be closed (not put back into connection pool) when the transaction finished.  This could handle unexpected bursts of transactions (with a performance penalty), but it also resulted in excessive socket opens/closes and too many sockets in a CLOSED/WAIT state.

Version 3.2.0 renames `ClientPolicy.maxThreads` to `ClientPolicy.maxConnsPerNode` and enforces a strict limit on maximum number of connections allowed per server node.  Synchronous transactions will now fail with `ResultCode.NO_MORE_CONNECTIONS` (after going through retry logic) if the maximum number of connections would be exceeded.

The number of connections used per node depends on how many concurrent threads issue database commands plus sub-threads used for parallel multi-node commands (batch, scan, and query). One connection will be used for each thread.

####**Version 3.1.4 - New Aerospike Server double type**

Aerospike Server versions >= 3.6.0 can store 64-bit double floating point values natively.  Previous server versions did not recognize a double data type, so clients sent doubles as long bits instead.  The java client can now send double as the new server double type when the following flag is set:

```java
	Value.UseDoubleType = true;
```

If the flag is false, double values will be sent as long bits. The flag currently defaults to false because both Aerospike Server and XDR (if used) must also be upgraded to versions >= 3.6.0 before the client uses the double data type.

The client can handle both return types (double or long) when receiving data from the server using Record.getDouble():

```java
	double val = record.getDouble("bin");
```

####**Version 3.1.0 - Bin constructors for list/map**

Two new Bin constructors have been added:

```java
	public Bin(String name, List<?> value)
	public Bin(String name, Map<?,?> value)
```

These constructors serialize list/map using a custom format supported natively by Aerospike 3 servers.  These constructors duplicate old static initializers which have now been marked deprecated.

```java
	@Deprecated
	public static Bin asList(String name, List<?> value)
	@Deprecated
	public static Bin asMap(String name, Map<?,?> value)
```

Previous code that wrote list/map may have used the default Bin object constructor:

```java
	public Bin(String name, Object value)
```

This object constructor serializes list/map using the java's default serializer format.  The new custom format is faster and can be introspected by the server, but the user may still want to preserve the old default java serialization (Running with Aerospike 2 servers is one reason).  If so, the following code modification is necessary:

Old:
```java
    new Bin(name, list)
    new Bin(name, map)
```
New:
```java
    new Bin(name, (Object)list)
    new Bin(name, (Object)map)
```

Finally, the underlying list/map Value methods have been modified as well:

```java
    public static Value get(List<?> value)
    public static Value get(Map<?,?> value)

	@Deprecated
	public static Value getAsList(List<?> value)
	@Deprecated
	public static Value getAsMap(Map<?,?> value)
```


####**Version 3.0.34 - Integer Bin Value Casting**

Always return 64 bit integers (long) for numbers retrieved from the server.  Code that casts record values to Integer must be changed to call "Record.getInt()" which performs the proper casting.

Old:
```java
    int value = (Integer)record.getValue("binname");
```
New:
```java
    int value = record.getInt("binname");
```
