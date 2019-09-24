---
title: Manage Indexes
description: Use the Aerospike Go client to manage secondary indexes.
---

Use the Aerospike Go client to create and delete secondary indexes in the database.

### Create a Secondary Index

To create a secondary index, invoke `Client.CreateIndex()`. Secondary indexes are created asynchronously, so the method can return before the secondary index has propagated across the cluster. The user can wait for the asynchronous server task to complete. 

This example creates an index and waits for index creation to complete. The example uses index _idx\_foo\_bar\_baz_ on namespace _foo_ within set _bar_ and bin _baz_.

```go
func (clnt *Client) CreateIndex(
    policy *WritePolicy,
    namespace string,
    setName string,
    indexName string,
    binName string,
    indexType IndexType,
) (*IndexTask, error)
```

```go
task, err := client.CreateIndex(null, "foo", "bar", 
    "idx_foo_bar_baz", "baz", aerospike.NUMERIC)

// to block until the Index is created
for err := range task.OnComplete(); err != nil {
    // deal with the error here
}
// task is completed successfully
```

- Only create secondary indexes once on the server. 
- Create secondary indexes using a combination of namespace, set, and bin name on either integer or string data types. 
  For example, if a secondary index is defined to index bin _x_ that contains integer values, then only records with bins _x_ containing integer values are indexed. Records containing bin _x_ with non-integer values are not indexed.
- When an index management call is made to any node in the Aerospike cluster, the information automatically propagates to remaining nodes.

### Removing a Secondary Index

To remove a secondary index using `Client.DropIndex()`:

```go
func (clnt *Client) DropIndex(
    policy *WritePolicy,
    namespace string,
    setName string,
    indexName string,
) error
```

