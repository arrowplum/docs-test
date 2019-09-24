---
title: Manage Indexes
description: The Aerospike Rust client provides the ability to create and delete secondary indexes.
---

The Aerospike Rust client allows you to create and delete secondary indexes
from the database.

### Creating a Secondary Index

To create a secondary index, invoke `Client.create_index()`. Secondary
indexes are created asynchronously, so the method returns before the secondary
index propagates to the cluster.

The following example creates an index *idx_foo_bar_baz*. The index is in
namespace _foo_ within set _bar_ and bin _baz_:

```rust
pub fn create_index(&self,
                    policy: &WritePolicy,
                    namespace: &str,
                    set_name: &str,
                    bin_name: &str,
                    index_name: &str,
                    index_type: IndexType)
                    -> Result<()>;
```

```rust
match client.create_index(&WritePolicy::default(), "foo", "bar", "baz",
    "idx_foo_bar_baz", IndexType::Numeric) {
    Err(err) => println!("Failed to create index: {}", err),
    _ => {}
}
```

Secondary indexes can only be created once on the server as a combination of
namespace, set, and bin name with either integer or string data types. For
example, if you define a secondary index to index bin _x_ that contains integer
values, then only records containing bins named _x_ with integer values are
indexed. Other records with a bin named _x_ that contain non-integer values are
not indexed.

When an index management call is made to any node in the Aerospike Server
cluster, the information automatically propagates to the remaining nodes.

### Removing a Secondary Index

To remove a secondary index using `Client.drop_index()`:

```rust
pub fn drop_index(&self,
                  policy: &WritePolicy,
                  namespace: &str,
                  set_name: &str,
                  index_name: &str)
                  -> Result<()>;
```
