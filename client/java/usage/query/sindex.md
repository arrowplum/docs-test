---
title: Manage Indexes
description: The Aerospike Java client provides the ability to create and delete secondary indexes.
---

The Aerospike Java client allows you to create and delete secondary indexes from the database.

### Creating a Secondary Index

To create a secondary index, invoke `AerospikeClient.createIndex()`. Secondary indexes are created asynchronously, so the method returns before the secondary index propagates to the cluster. As an option, the client can wait for the asynchronous server task to complete. 

The following example creates an index *idx_foo_bar_baz* and waits for index creation to complete. The index is in Namespace _foo_ within set _bar_ and bin _baz_:

```java
public class AerospikeClient {
    public final IndexTask createIndex(
        Policy policy,
        String namespace,
        String setName,
        String indexName,
        String binName,
        IndexType indexType
    )   throws AerospikeException
}
```

```java
IndexTask task = client.createIndex(null, "foo", "bar", 
    "idx_foo_bar_baz", "baz", IndexType.NUMERIC);

task.waitTillComplete();
```

Secondary indexes can only be created once on the server as a combination of Namespace, set, and bin name with either integer or string data types. For example, if you define a secondary index to index bin _x_ that contains integer values, then only records containing bins named _x_ with integer values are indexed. Other records with a bin named _x_ that contain non-integer values are not indexed.

When an index management call is made to any node in the Aerospike Server cluster, the information automatically propagates to the remaining nodes.

### Removing a Secondary Index

To remove a secondary index using `AerospikeClient.dropIndex()`:

```java
public class AerospikeClient {
    public final void dropIndex(
        Policy policy,
        String namespace,
        String setName,
        String indexName
    )   throws AerospikeException
}
```

