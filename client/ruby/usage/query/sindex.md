---
title: Managing Indexes
description: The Aerospike Ruby client provides the ability to create and delete secondary indexes.
---

The Aerospike Ruby client provides the ability to create and delete secondary indexes.

### Creating a Secondary Index

To create a secondary index, invoke `Client.create_index`. Secondary indexes are created asynchronously, so the method returns before the secondary index propagates to the cluster. As an option, the client can wait for the asynchronous server task to complete. 

The following example creates an index *idx_ns_set_bin* and waits for index creation to complete. The `idx_ns_set_bin` index is in namespace _ns_ within set _set_ and bin _bin_:


```ruby
task = client.create_index('ns', 'set', 'idx_ns_set_bin', 'bin', :numeric)

# to block until the Index is created
task.wait_till_completed
# task is completed successfully
```

Secondary indexes can only be created once on the server as a combination of Namespace, set, and bin name with either integer or string data types. For example, if you define a secondary index to index bin _x_ that contains integer values, then only records containing bins named _x_ with integer values are indexed. Other records with a bin named _x_ that contain non-integer values are not indexed.

When an index management call is made to any node in the Aerospike Server cluster, the information automatically propagates to the remaining nodes.

### Removing a Secondary Index

To remove a secondary index, invoke `Client.DropIndex()`:

```ruby
client.drop_index('ns', 'set', 'idx_ns_set_bin')
```

