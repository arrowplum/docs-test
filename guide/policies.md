---
title: Policies
description: Use Aerospike clients to create policies for use with the Aerospike database. 
---

Aerospike allows reading and writing with great flexibility. With an Aerospike client `policy`,
you can create read-modify-write patterns of optimistic concurrency, control the time to live, and write
a record only if no record previously exists (or the converse).

Operations of this type are very quick, because information like the generation and time-to-live are stored
in the primary key index. No extra work is needed to retrieve the data object.

These policies affect both database operations and client operations. Many policies are used to send the
appropriate wire protocol commands to the server.  Other policies (like maxRetries) affect client operation. 

These policies exist with each client, and have slightly different APIs. After understanding which policies
you need for your application, please see the client specific documentation for precise syntax.

### Set Default Client Policies

Default client policies can be created for each AerospikeClient instance.  The following example demonstrates
how to set policy defaults in the Java client.  For language-specific examples, see the documentation for your
client.

```java
// Set client default policies.
ClientPolicy clientPolicy = new ClientPolicy();
clientPolicy.readPolicyDefault.replica = Replica.MASTER_PROLES;
clientPolicy.readPolicyDefault.consistencyLevel = ConsistencyLevel.CONSISTENCY_ALL;
clientPolicy.readPolicyDefault.socketTimeout = 100;
clientPolicy.readPolicyDefault.totalTimeout = 100;
clientPolicy.writePolicyDefault.commitLevel = CommitLevel.COMMIT_ALL;
clientPolicy.writePolicyDefault.socketTimeout = 500;
clientPolicy.writePolicyDefault.totalTimeout = 500;

// Connect to the cluster.
AerospikeClient client = new AerospikeClient(clientPolicy, new Host("seed1", 3000));
```

### Set Per-Transaction Client Policies

To set policies on a per-transaction basis, pass the desired policy settings to the individual API call.
For example, to perform writes with the `master` commit level:

```cpp
// Make a copy of the client's default write policy.
WritePolicy policy = new WritePolicy(client.writePolicyDefault);

// Change commit level.
policy.commitLevel = ConsistencyLevel.COMMIT_MASTER;

// Write record with modified write policy.
client.put(policy, key, bins);
```

If the policy specified in the transaction call is not null, that policy overrides the corresponding policy
defined at the client connection level.  If the transaction policy is null, the corresponding default client
policy will be used.

### Server Policies

You can override client-selected per-transaction data consistency levels on the server using dynamically
changeable server configuration parameters at the namespace level.

Refer to the consistency override configuration parameters in the reference page: [read-consistency-level-override](/docs/reference/configuration#read-consistency-level-override) and [write-commit-level-override](/docs/reference/configuration#write-commit-level-override).

### Policy Definitions

The following section describes the Aerospike Java client policies.  Other clients use similar constructs.

#### Replica

Replica (`Policy.replica`) specifies which replica the client will access during the single record
operation:

- `SEQUENCE` (default) &mdash; Try node containing key's master partition first.  If connection fails, all commands try nodes containing replicated partitions.  If socketTimeout is reached, reads also try nodes containing replicated partitions, but writes remain on master node.
- `MASTER` &mdash; Use node containing key's master partition.
- `MASTER_PROLES` &mdash; Distribute reads across nodes containing key's master and replicated partitions in round-robin fashion.  Writes always use node containing key's master partition.
- `RANDOM` &mdash; Distribute reads across all nodes in cluster in round-robin fashion.  Writes always use node containing key's master partition.

By default, all client reads are first directed to the master replica (`Replica.SEQUENCE`), however, you may
want to spread the reads over all available replicas (for example, the performance impact of reading a hot key
can be reduced along the order of the replication factor). Set the replica policy to `Replica.MASTER_PROLES`
to distribute reads across master and proles.

#### Data Consistency Level

Consistency level (`Policy.consistencyLevel`) specifies how many replicas the server is to consult
internally to determine the most-recent record value, and return it to the client:

- `CONSISTENCY_ONE` (default) &mdash; Read a single replica before returning.
- `CONSISTENCY_ALL` &mdash; Read all replicas before returning.

The default client behavior when reading a record (including `read` and `operate` functions) is to read only
one replica (`ConsistencyLevel.CONSISTENCY_ONE`). 

During cluster reconfiguration, reading a single replica may not return the most recently-written version.
If you wish the server to provide "duplicate resolution", which is, to contact replicas and find the most
recent version, including updating the master's copy, set the consistency level policy to
`ConsistencyLevel.CONSISTENCY_ALL`.

{{#note}}
The potential performance degradation due to reading all replicas is only significant during cluster
reconfiguration.
{{/note}}

#### Linearize Read

If enabled, the linearize read policy (`Policy.linearizeRead`) forces reads to be linearized for server namespaces
that support strong consistency mode.

#### Send Key

If enabled, send key (`Policy.sendKey`) sends the user-defined key in addition to hash digest on both reads and
writes.  If the key is sent on a write, the key will be stored with the record on the server and returned to the
client on scans and queries.

#### Socket Timeout

Socket timeout (`Policy.socketTimeout`) specifies socket idle timeout in milliseconds when processing a
database command.

If socketTimeout is not zero and the socket has been idle for at least socketTimeout, both maxRetries and
totalTimeout are checked.  If maxRetries and totalTimeout are not exceeded, the transaction is retried.

If both socketTimeout and totalTimeout are non-zero and socketTimeout > totalTimeout, then socketTimeout
will be set to totalTimeout.

If socketTimeout is zero, there will be no socket idle limit.

#### Total Timeout

Total timeout (`Policy.totalTimeout`) specifies total transaction timeout in milliseconds.

The totalTimeout is tracked on the client and sent to the server along with the transaction in the wire protocol.
The client will most likely timeout first, but the server also has the capability to timeout the transaction.

If totalTimeout is not zero and totalTimeout is reached before the transaction completes, the transaction will
abort with a timeout exception.

#### Max Retries

Max retries (`Policy.maxRetries`) specifies the maximum number of retries before aborting the current
transaction.  The initial attempt is not counted as a retry.

If maxRetries is exceeded, the transaction will abort with a timeout exception.

{{#note}}
Database writes that are not idempotent (such as add()) should not be retried because the write operation may
be performed multiple times if the client timed out previous transaction attempts.  It's important to use a
distinct WritePolicy for non-idempotent writes which sets maxRetries to zero.
{{/note}}

Default for read: 2 (initial attempt + 2 retries = 3 attempts)

Default for write/query/scan: 0 (no retries)

#### Sleep Between Retries

Sleep between retries (`Policy.sleepBetweenRetries`) is the milliseconds to sleep between retries.
Enter zero to skip sleep.  This field is ignored when maxRetries is zero.  This field is also ignored in async
mode.

The sleep only occurs on connection errors and server timeouts which suggest a node is down and the cluster is
reforming.  The sleep does not occur when the client's socketTimeout expires.

Reads do not have to sleep when a node goes down because the cluster does not shut out reads during cluster
reformation.  The default for reads is zero.

The default for writes is also zero because writes are not retried by default.  Writes need to wait for the
cluster to reform when a node goes down.  Immediate write retries on node failure have been shown to consistently
result in errors.  If maxRetries is greater than zero on a write, then sleepBetweenRetries should be set high
enough to allow the cluster to reform (>= 500ms).

#### Write Mode

The write mode (`WritePolicy.recordExistsAction`) specifies how to handle writes where the record already
exists.

- `UPDATE` (default) &mdash; Create or update record.  Merge write command bins with existing bins.
- `UPDATE_ONLY` &mdash; Update record only.  Fail if record does not exist.  Merge write command bins with existing bins.
- `REPLACE` &mdash; Create or replace record.  Delete existing bins not referenced by write command bins.
- `REPLACE_ONLY` &mdash; Replace record only.  Fail if record does not exist.  Delete existing bins not referenced by write command bins.
- `CREATE_ONLY` &mdash; Create only.  Fail if record exists.

#### Write Commit Level

The commit level policy (`WritePolicy.commitLevel`) specifies how many replicas the server must write successfully before successfully returning to the client:

- `COMMIT_ALL` (default) &mdash; Commit all replicas before returning.
- `COMMIT_MASTER` &mdash; Return after committing only the master replica and replicate the `prole` replica(s) asynchronously.

The default client behavior when modifying a record (including `write`, `remove`, `operate`, and UDF functions) is to confirm that all replicas were successfully written before returning success from the write-related API. This default policy (`CommitLevel.COMMIT_ALL`) provides the highest level of write consistency.

If a lower write latency is desired and the application can tolerate a lower write consistency level (with the possibility of 'dirty reads,' which is when an older value returns if a read of the same record is done from a non-master replica before the replica is committed), set the commit level policy to `CommitLevel.COMMIT_MASTER`:

#### Write Generation Policy

The generation policy (`WritePolicy.generationPolicy`) specifies how to handle record writes based on record generation.

- `NONE` (default) &mdash; Do not use record generation to restrict writes.
- `EXPECT_GEN_EQUAL` &mdash; Update/delete record if expected generation is equal to server generation. Otherwise, fail.
- `EXPECT_GEN_GT` &mdash; Update/delete record if expected generation greater than the server generation. Otherwise, fail.  This is useful for restore after backup.

#### Expiration (Time To Live)

Record expiration (`WritePolicy.expiration`) or time to live (ttl) is the number of seconds the record will live before
being removed by the server.  Expiration values:

* -2 &mdash; Do not change ttl when record is updated.
* -1 &mdash; Never expire.
* 0 &mdash; Default to namespace configuration variable "default-ttl" on the server.
* &gt; 0 &mdash; Actual ttl in seconds.

#### Durable Delete

If enabled, durable delete (`WritePolicy.durableDelete`) leaves a tombstone when a record is deleted.  This prevents deleted records from reappearing after node failures.
