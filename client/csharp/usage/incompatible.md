---
title: Incompatible API Changes
description:  Compatibility issues with the Aerospike C# client.
---

####**Version 3.8.0 - Read consistency level changes**

**Read policy changes for transactions involving AP (availability) namespaces**

- Policy variable `ConsistencyLevel consistencyLevel` moved to `ReadModeAP readModeAP`.

**Read policy changes for transactions involving SC (strong consistency) namespaces**

- Policy variable `bool linearizeRead` moved to `ReadModeSC readModeSC`.
- Add `ReadModeSC` enum below:

```cs
/// <summary>
/// Read policy for SC (strong consistency) namespaces.
/// <para>
/// Determines SC read consistency options.
/// </para>
/// </summary>
public enum ReadModeSC
{
	/// <summary>
	/// Ensures this client will only see an increasing sequence of record versions.
	/// Server only reads from master.  This is the default.
	/// </summary>
	SESSION,

	/// <summary>
	/// Ensures ALL clients will only see an increasing sequence of record versions.
	/// Server only reads from master.
	/// </summary>
	LINEARIZE,

	/// <summary>
	/// Server may read from master or any full (non-migrating) replica.
	/// Increasing sequence of record versions is not guaranteed.
	/// </summary>
	ALLOW_REPLICA,

	/// <summary>
	/// Server may read from master or any full (non-migrating) replica or from unavailable
	/// partitions.  Increasing sequence of record versions is not guaranteed.
	/// </summary>
	ALLOW_UNAVAILABLE
}
```


####**Version 3.7.0 - Remove obsolete code**

The following old obsoleted code has been removed.

- `Statement.Filters` property:  This property was replaced by the `Statement.Filter` property.  The server accepts only one filter.

- `Value.UseDoubleType`: This configuration variable is no longer necessary because all server versions that this client supports also support the double type.  This client supports server versions >= 3.6.0.

- `ClientPolicy.requestProleReplicas`:   This configuration variable is no longer necessary because prole replicas are now always requested.

- Batch direct protocol code: This old batch direct protocol code was replaced by the batch index functionality.  This code was no longer accessible by the external API.

### **Version 3.6.8 - Remove BatchPolicy.useBatchDirect**

`BatchPolicy.useBatchDirect` was removed because the server has removed support for the old batch direct protocol.  Remove any references to this policy field.

### **Version 3.5.0 - Policy Changes, LDT Removal and Index Changes**

#### Policy Changes

The **[Policy](https://github.com/aerospike/aerospike-client-csharp/blob/master/Core/AerospikeClient/Policy/Policy.cs)** class has been redefined to support greater control on timeout behavior and provide defaults that improve retry success.

**Split `timeout` into `socketTimeout` and `totalTimeout`**

socketTimeout is the socket idle timeout in milliseconds.  For synchronous methods, socketTimeout is the socket's SendTimeout and ReceiveTimeout.  For asynchronous methods, the socketTimeout is implemented using an AsyncTimeoutQueue and socketTimeout is only used if totalTimeout is not defined.

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

#### LDT Removal

LDT methods have been removed on client because large data types are no longer supported by Aerospike Server.

#### Index Changes

Index exception handling has changed on the client.  In the past, the client swallowed INDEX_ALREADY_EXISTS errors on CreateIndex().  The client now throws an AerospikeException when an index already exists on CreateIndex().

The client also used to swallow INDEX_NOTFOUND errors on DropIndex().  The client now throws an AerospikeException when an index does not already exist on DropIndex().

To preserve old behavior, calling code can be changed to:

```cs
try {
    IndexTask task = client.CreateIndex(policy, ns, set, indexName, binName, indexType);
    task.Wait();
}
catch (AerospikeException ae) {
    if (ae.getResultCode() != ResultCode.INDEX_ALREADY_EXISTS) {
        throw ae;
    }
}

try {
	client.DropIndex(policy, ns, set, indexName);		
}
catch (AerospikeException ae) {
    if (ae.getResultCode() != ResultCode.INDEX_NOTFOUND) {
        throw ae;
    }
}
```

### **Version 3.2.0 - Max connections is a now a hard limit**

Past versions used to create new connections when a node's outstanding connections exceeded `ClientPolicy.maxThreads`.  This connection would then be closed (not put back into connection pool) when the transaction finished.  This could handle unexpected bursts of transactions (with a performance penalty), but it also resulted in excessive socket opens/closes and too many sockets in a CLOSED/WAIT state.

Version 3.2.0 renames `ClientPolicy.maxThreads` to `ClientPolicy.maxConnsPerNode` and enforces a strict limit on maximum number of connections allowed per server node.  Synchronous transactions will now fail with `ResultCode.NO_MORE_CONNECTIONS` (after going through retry logic) if the maximum number of connections would be exceeded.

The number of connections used per node depends on how many concurrent threads issue database commands plus sub-threads used for parallel multi-node commands (batch, scan, and query). One connection will be used for each thread.

`ClientPolicy.maxConnsPerNode` is ignored by asynchronous transactions since these transactions are already bound by asyncMaxCommands by default. Each async command has a one-to-one relationship with a connection.

### **Version 3.1.4 - New Aerospike Server double type**

The Aerospike server version 3.6.0 and later can store 64-bit double floating point values natively.

Previous server versions did not recognize the double data type, so clients sent doubles as long bits. The client can now send double as the new server double type when the following flag is set:

```
	Value.UseDoubleType = true;
```

If `Value.UseDoubleType = false` (default), double values are sent as long bits. `Value.UseDoubleType` defaults to false because the Aerospike server and XDR (if used) must both upgrade to version 3.6.0 or later before the client can use the double data type.

The client can handle both return types (double or long) when receiving data from the server using `Record.GetDouble()`:

```
	double val = record.GetDouble("bin");
```


### **Version 3.1.1 - Bin Constructors for `list`/`map`** 

Two bin constructors were added:

```
	public Bin(string name, IList value)
	public Bin(string name, IDictionary value)
```

These constructors serialize `list`/`map` using a custom format supported natively by the Aerospike server. These constructors duplicate old static initializers, which are deprecated.

```
	// Deprecated
	public static Bin AsList(string name, IList value)
	// Deprecated
	public static Bin AsMap(string name, IDictionary value)
```

Previous code that wrote `list`/`map` may have used the default `Bin` object constructor:

```
	public Bin(string name, object value)
```

This object constructor serializes `list`/`map` using the C# default serializer format. The custom format is faster and can be introspected by the server, but to preserve the old default C# serialization (for example, if you're running with Aerospike 2 servers), modify the code:

**Old**
```
    new Bin(name, list)
    new Bin(name, map)
```
**New**
```
    new Bin(name, (object)list)
    new Bin(name, (object)map)
```

Finally, the underlying `list`/`map` value methods were also modified:

```
    public static Value Get(IList value)
    public static Value Get(IDictionary value)

	// Deprecated
	public static Value GetAsList(IList value)
	// Deprecated
	public static Value GetAsMap(IDictionary value)
```


### **Version 3.1.0 - NeoLua Replacement**

NeoLua uses the Lua 5.3 specification, which differs slightly from scripts written for Lua 5.1. The most relevant change is that NeoLua tries to store Lua numbers as 64-bit integers (long) when the number has no decimal component.  When retrieving integers from C# aggregation queries (which use Lua), previous (double) casts must be changed to (long) casts. 

Also, the server still uses the Lua 5.1 specification. Scripts must be written to comply with both Lua 5.1 and 5.3, until the server is upgraded to Lua 5.3.
