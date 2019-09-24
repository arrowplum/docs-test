---
title: Consistency
description: Aerospike Strong Consistency uses primary key guarentees and provides no-loss write guarentees.
assets: /docs/guide/assets
---
<div style="float: right" >
![]({{book.assets}}/icon-query-2.png)
</div>


Strong Consistency guarantees that all writes to a single record will be applied in a 
specific order (sequentially), and writes will not be re-ordered or skipped, ie, they will not be lost.

In all forms of Strong Consistency, writes will not be lost - within the bounds of simultaneous hardware failures.
For the exact statements on which hardware failures can cause lost data, please see our 
[Consistency Architecture Guide](/docs/architecture/consistency.html).

Aerospike guarantees that data will not be lost, with three exceptions:

1. Node process pauses greater than 27 seconds, or clock skew between cluster nodes of greater than 27 seconds.

1. Simultaneous (within the [`flush-max-ms`](/docs/reference/configuration#flush-max-ms)) unclean or untrusted server shutdowns if [`commit-to-device`](/docs/reference/configuration#commit-to-device) is not enabled.

1. Multiple hardware storage failures before data can replicate.

In each case, Aerospike attempts to provide the best availability and minimize potential issues.

In the case of clock skew, Aerospike's gossip cluster protocol continually monitors the amount of skew, and will
warn if the skew becomes large - and disables the cluster before data loss would occur.

In the case of server crashes or shutdowns, Aerospike automatically rapidly replicates and rebalances data
in the case of the first failure. If the failures occur rapidly, then sections of the data may be marked
"dead", and require operator intervention. This is to allow read-only use of the data, or allow reloading of recent changes.

## Configuring for Strong Consistency

Strong Consistency is enabled on an entire namespace. [The Consistency Management documentation](/docs/operations/configure/consistency/index.html)
explains how to configure a namespace and a roster.

## Managing Strong Consistency

Managing Strong Consistency is more complex than managing Available namespaces.

 [This section](/docs/operations/manage/consistency/index.html) describes adding and removing nodes, 
 starting and stopping servers safely, and other management concepts.

## Client Requirements

Only the Aerospike Java client 4.1.2, C client 4.3.5, C# 3.5.3, and Python 3.0.1 releases support Strong Consistency. 

Using a previous client which doesn't support "strong-consistency" will operate, however; you may get stale reads, 
and you may generate consistency violations ( lost intermediate state ) due to retransmissions. 
If you use a legacy client version, you will not lose writes but you may get neither 
session consistency nor linearizability.

Versions slightly before those stated may have the correct APIs but have a variety of resolved bugs 
regarding return codes and stale node state. Please use the stated versions, or better.

Please see the subsequent sections regarding new API functionality related to Strong Consistency.

## Upgrading from AP to SC namespaces

In general, changing from AP to SC namespaces is not supported. There are cases where consistency 
violations may occur during the change. Creating a new namespace, then backing up and restoring, 
is recommended ( "forklift" upgrade procedure ).

## Using Strong Consistency

In general, there are few changes as a programmer. Primarily, you simply know that data is safe.

However, there are some new capabilities, managed through the client Policy object,
as well as differences in error code meanings.

### Linearizable Reads

A new field exists on the 'Policy' object. In order to attain Linearizable reads, you must set the  `linearizableRead`
field to `true`. If you set this field to a read on a non-SC configured namespace, the read will fail. 
If you do not set this field on an SC namespace, you will get Session Consistency. If you set this field on a 
request to a server before 4.0, the field will be ignored.

The `Policy` object can be set in the initial AerospikeClient constructor call. 
All subsequent calls using the constructed AerospikeClient object will, by default, use that value. 
Otherwise, you should use this constructed Policy object on individual operations to denote whether 
to execute a fully Linearized read, or a Session Consistency read.

Several Policy objects - BatchPolicy, WritePolicy, QueryPolicy, and ScanPolicy - inherit from the Policy object. 
Of these, the new 'linearizableRead' field only makes sense for the default object and the inherited 
BatchPolicy object, where it is applied for all elements in the batch operation.

### InDoubt Errors

A new field on all error returns called `InDoubt` has been added. This is to denote the difference where a 
write has _certainly_ not been applied, or _may_ have been applied. 

In most database APIs ( such as the SQL standard ), failures such as TIMEOUT are "known" to be uncertain, 
but there is no specific flag to denote which errors are uncertain. 
Common practice is to read the database to determine if a writes has been applied.

For example, if the the client driver timed out before contacting a server, the client driver 
may be certain that the transaction was not applied; if the client has attached to a server and sent 
the data but receives no response over TCP, the client is unsure whether the write has been applied. 
The Aerospike API improvement allows the client to denote which 
failures have _certainly_ not been applied.

We believe this flag can be useful for the advanced programmer and reduce cases where 
an error requires reading from the database under high stress situations.

### Data Unreachable Errors

In SC, there will be periods where data is unavailable because of network partition or other outage. 
There are several errors the client could present if data is, or becomes, unavailable.

These errors already exist in Aerospike, but have important distinctions when running with Strong Consistency. Here are the list of errors for some 
of the main Client Libraries:

- [Java Client Error Code](https://www.aerospike.com/apidocs/java/com/aerospike/client/ResultCode.html).

- [C# Client Error Code](https://www.aerospike.com/apidocs/csharp/html/AllMembers_T_Aerospike_Client_ResultCode.htm).

- [C Client Error Code](https://www.aerospike.com/apidocs/c/dc/d42/as__status_8h.html).


Four different errors can be seen when a cluster has partitioned, or has multiple hardware failures resulting 
in data unavailability (using here the Java Client error codes): `PARTITION_UNAVAILABLE`, `INVALID_NODE_ERROR`, `TIMEOUT` and `CONNECTION_ERROR`. 

{{#note}}
Other clients may have different error codes for those conditions. Refer to the Client Library specific error code tables for details.
{{/note}}

The error `PARTITION_UNAVAILABLE` is returned from the server when that server determines that its cluster 
does not have the correct data. In specific, the client can connect to a cluster, and has received a 
partition table that includes this partition data initially. In this case, the client will send a 
request to the most recent server it has  heard from. If that server is part of a cluster that no 
longer has data availability, the `PARTITION_UNAVAILABLE` error will be returned.

The error `INVALID_NODE_ERROR` is generated by the client in cases where the client does not have 
a node to send a request to. This happens initially when the specified "seed node" addresses are incorrect, 
when the client can't connect to the particular node(s) in the list of seeds and thus never receives a full partition map, 
and also when the partition map received from the server does not contain the partition in question.

`INVALID_NODE_ERROR` will also occur if the roster has been mis-configured, or if the 
server has [`dead_partitions`](/docs/reference/metrics/?show-removed=1#dead_partitions) or 
[`unavailable_partitions`](/docs/reference/metrics/?show-removed=1#unavailable_partitions) (and needs maintenance or re-clustering). 
If you get this error, please validate that data in the cluster is available using the steps above.

Two other errors, `TIMEOUT` and `CONNECTION_ERROR`, will be returned in cases where a network partition has 
happened or is happening. The client must have a partition map and thus a node to send the request to, 
but can't connect to that node, or has connected but subsequently the failure occurs. The `CONNECTION_ERROR`
may persist as long as the network partition, and only goes away when the partition has been healed or operator 
intervention has changed the cluster rosters.

## Use of AQL with Strong Consistency

The AQL tool is commonly used by developers and administrators getting started with Aerospike. There are several command you should be aware of regarding SC use of AQL.
In order to use AQL, please make sure you are running Aerospike Tools release 3.15.3.2 or newer.

### AQL Durable Delete

In order to delete data from an SC namespace, you will almost certainly wish to use a durable delete. 
Expunge deletes ( which recover storage immediately at the cost of consistency ) are not allowed by default, 
and will result in a FORBIDDEN error. However, these deletes are the default in AQL.

In order to have AQL generate durable deletes:
```
aql> set DURABLE_DELETE true
```

This will cause subsequent deletes in the same AQL session to be executed with the delete flag.

### AQL Linearize

In order to have reads ( ie, SELECT in AQL ) be executed with the Linearize flag: 
```
aql> set LINEARIZE_READ true
```

All subsequent primary key select commands will be executed with Linearize read policy.

Please note that Linearize is only supported on primary key requests, not on queries and scans as noted in this 
document, thus will only apply when using the "PK =" SELECT form.

### AQL Error codes

As AQL is often used in initial use to diagnose and validate a configuration, please understand the 
distinction between the error codes: `PARTITION_UNAVAILABLE`, `INVALID_NODE_ERROR`, `TIMEOUT`, and `CONNECTION_ERROR`. 

These error codes are described in depth above.

## Consistency Guarantees

### Strong Consistency

The strong consistency guarantee states that all writes to a single record will be applied in a specific 
order (sequentially), and writes will not be re-ordered or skipped. 

In particular, writes that are acknowledged as committed have been applied, and exist in the transaction 
timeline in contrast to other writes to the same record. This guarantee applies even in the face of network 
failures and outages, and partitions. Writes which are designated as "timeouts" (or "InDoubt" from the client API) 
may or may not be applied, but if they have been applied they have been applied and only observed as such.

Aerospike's strong consistency guarantee is per-record, and involves no multi-record transaction 
semantics. Each record's write or update will be atomic and isolated, and ordering is guaranteed using a hybrid clock.

Aerospike provides both full Linearizable mode, which provides a single linear view among 
all clients that can observe data, as well as a more practical Session Consistency mode, 
which guarantees an individual process sees the sequential set of updates. These two read policies can 
be chosen on a read-by-read basis, thus allowing the few transactions that require a higher guarantee 
to pay the extra synchronization price, and are detailed below. 

In the case of a "timeout" return value - which could be generated due to network congestion, 
external to any Aerospike issue - the write is guaranteed to be written, or not written.

Strong Consistency is configured on a per-namespace basis. Switching a namespace from one 
mode to another is impractical - creating a new namespace and migrating data is the recommended means.

### Linearizability

In concurrent programming, an operation (or set of operations) is atomic, linearizable, 
indivisible or uninterruptible if it appears to the rest of the system to occur instantaneously - 
or is made available for reads instantaneously. 

All accesses are seen by all parallel processes in the same order (sequentially). 
This guarantee is enforced by the Aerospike effective master for the record, and results in an 
in-transaction "health check" to determine the state of the others servers. 

If a write is applied and observed to be applied by a client, no prior version of the record 
will be observed. With this client mode enabled, "global consistency" - referring to all clients 
attaching to the cluster - will see a single view of record state at a given time.

This mode requires extra synchronization on every read, thus incurs a performance penalty. 
Those synchronization packets do not look up or read individual records, but instead simply 
validate the existence and health of individual partition.

### Session Consistency

This mode, called in other databases by the names "Monotonic reads, monotonic writes, 
read-your-writes, write-follows-reads", is the most practical of strong consistency modes.

Unlike the Linearizable model, session consistency is scoped to a client session - which 
in this case is an Aerospike cluster object on an individual client system, unless shared 
memory is used to share cluster state between processes.

Session consistency is ideal for all scenarios where a device or user session is involved 
since it guarantees monotonic reads, monotonic writes, and read your own writes (RYW) guarantees.

Session consistency provides predictable consistency for a session, and maximum read 
throughput while offering the lowest latency writes and reads.

## Performance Considerations

Strong Consistency mode is similar to Availability mode in performance when used with the following settings:

1. Replication factor two,
1. Session Consistency.

When the replication factor is more than 2, a write causes an extra "replication advise" packet to acting replicas. 
While the master does not wait for a response, the extra network packets will create load on the system.

When the Linearizability read concern is enabled, during a read the master must send a request 
to every acting replica. These "regime check" packets - which do not do full reads - cause extra latency and packet load, and decrease performance.

## Availability Considerations

Although Aerospike allows operation with two copies of the data, availability in a 
failure case requires a replica during a master promotion, which then requires during the course of a failure three copies ( the copy that failed, the new master, and the prospective replica). Without this third potential copy, a partition may remain unavailable. For this reason, a two node cluster - with two copies of the data - will not be available in a split.

## Exceptions to Consistency Guarantees

Please be aware of the following operational circumstances which could cause issues with consistency or durability.

### Clock discontinuity

A clock discontinuity of more than approximately 27 seconds may result in lost writes. In this case, 
data is written to storage with a time value outside the 27 second clock resolution. A subsequent data merge 
may pick this uncommitted write over other successfully completing versions of the record, due to 
the precision of the stored clock values.

In the case where the discontinuity is less than 27 seconds, the resolution of the internal hybrid clock 
will choose the correct record. 

Organic clock skew is not an issue, the cluster's heartbeat mechanism detects the drift (at a skew of 15 
seconds by default), and logs warnings. If the skew becomes extreme (20 seconds by default), 
the node rejects writes, returning the "forbidden" error code, thus preventing consistency violations.

Clock discontinuities of this type tend to occur due to four specific reasons:

1. Administrator Error, where an operator executes setting the clock far into the future or far into the past
1. Malfunctioning time synchronization components
1. Hibernation of virtual machines
1. Linux process pause or Docker container pause

Please be certain to avoid these issues.

### Clock discontinuity due to virtual machine live migrations

Virtual machines and virtualized environments, can result in safe strong consistency deployments.

However, two specific configuration issues are hazardous.

**Live migrations**, which is the process of moving a virtual machine from one physical chassis to 
another transparently, causes certain hazards. Clocks move with discontinuities, and network buffers
can remain in lower level buffers and be applied later.

If live migrations can be certainly limited to less than 27 seconds, strong consistency can be
maintained. However, the better operational process is to safely stop Aerospike, move the virtual machine,
then restart Aerospike.

The second case involves process pauses, which are also used in container environments. These create
similar hazards, and should not be executed. There are few operational reasons to use these features,

### UDFs

The UDF system will function, but, currently, UDF reads will not be linearized, and UDF writes that 
fail in certain ways may result in inconsistencies. 

### Non-durable deletes, expiration and data eviction

Non-durable deletes, including data expiration and eviction, are not consistent. 
These deletes have the necessary characteristic that they free DRAM and storage, 
so they may be required even with strong consistency.

These removals from the database which do not generate persistent "tombstones" may violate consistency guarantees. 
In these cases you may see data return. For this reason, we generally require disabling eviction and 
expiration in SC configurations, however, as there are valid use cases, we allow manual override.

However, there are cases where expiration and non-durable deletes may be required even for SC namespaces,
and may be known to be safe to the architects, programmers, and operations staff.

These are generally cases where a few objects are being modified, and "old" objects are being expired and deleted.
No writes are possible to objects which may be subject to expiration or deletion. 

For example, writes may occur to an object - which represents a financial transaction - for a few seconds, then 
the object may exist in the database for several days without writes. This object may safely be expired, as the period
of time between the writes and the expiration is very large.

If you are certain that your use case is suitable, you may enable non-durable deletes, please use the following 
configuration setting in your `/etc/aerospike/aerospike.conf` adding a namespace option:
```
strong-consistency-allow-expunge true
```

Durable deletes, which do generate tombstones, fully support consistency.


## Client version

In order to achieve Session Consistency or Linearizable consistency, you must use an Aerospike client version that 
supports these features. As of writing, current minimal client versions are 
Java 4.1.2 (and higher), C Client 4.3.5 or higher, C# 3.5.3  or Python 3.0.1 or higher.

If you use other client versions, you may see stale reads.


## Client retransmit

If you enable Aerospike client write retransmission, you may find that certain test suites will claim consistency 
violation. This is because an initial write results in a timeout, and is thus retransmitted, and potentially
applied multiple times.

Several hazards are present.

First, the write may be applied multiple times. This write may then "surround" other writes, causing consistency
violations. This can be avoided using the "read-modify-write" pattern and specifying a generation, as well as
disabling retransmission.

Second, incorrect error codes may be generated. For example, a write may be correctly applied but a transient
network fault might cause retransmission. On the second write, the disk may be full, generating a specific and
not "InDoubt" error - even though the transaction was applied. This class of error can't be resolved
by using the "read-modify-write" pattern.

We strongly recommend disabling the client timeout functionality, and retransmitting as desired in the application.
While this may seem like extra work, the benefits of correct error codes while debugging is invaluable.

## Secondary Index requests ( scan and query )

If a scan or query is executed, both stale reads and dirty reads may be returned. 
In the interest of performance, Aerospike will currently return data that is "In Doubt" on a master and has not 
been fully committed.

This violation only exists for queries and scans, and will be rectified in subsequent releases.

# Durability Exceptions

## Storage hardware failure

Some durability exceptions are detected, marking the partition as a `dead_partiton` in administrative interfaces.

This happens when all nodes within the entire roster are available and connected, yet some partition data is not 
available. To allow a partition to accept reads and writes again, an administrator must override the error by 
executing a "revive" command above, or bring a server online with the missing data.

These cases occur when Strong Consistency is used with a data in memory configuration with no 
backing store, or when backing storage is either erased or lost.


## Incorrect Roster Management

Reducing the roster by replication-factor or more nodes at a time may result in loss of record data. 
The procedure for safe addition and removal from a cluster must be followed. 
Please follow the operational procedures regarding roster management closely.

## Partial storage erasure

In the case where some number of sectors or portions of a drive have been erased by an operator, 
Aerospike will not be able to note the failure. Partial data erasure (e.g. malicious user or drive failure) 
on replication-factor or more nodes may erase records and escape detection.


## Simultaneous server restarts

By default, Aerospike writes data initially to a buffer, and considers the data written when all
the required server nodes have acknowledged receiving the data and placing it in the persistent queue.

Aerospike attempts to remove all other write queues from the system. The recommended Aerospike hardware
configuration uses a RAW device mode which disables all operating system page caches and most device caches. We
recommend disabling any hardware device write caches, which must be done in a device-specific fashion.

Aerospike will immediately begin the process of replicating data in case of a server failure. If replication occurs quickly,
no writes will be lost.

In order to protect against data loss in the write queue during multiple rapid failures, please
enable `commit-to-device` noted elsewhere. The default algorithms cause the data lost to be limited 
and provide the highest levels of performance, but this feature can be enabled on an individual 
storage-based namespace if the higher level of guarantee is required.

In the case where a filesystem or device with a write buffer is used as storage, `commit-to-device` may not prevent
these kind of buffer based data losses.




