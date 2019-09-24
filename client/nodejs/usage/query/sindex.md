---
title: Manage Indexes
description: Use the Aerospike Nodejs client to create and delete secondary indexes from the database.
---

Use the Aerospike Node.js client to create and delete secondary indexes from the database.

Aerospike supports dynamic creation of secondary indexes. Tools such as aql are available which read the indexes currently available, and allow creation and destruction of indexes. See [Secondary Index](/docs/architecture/secondary-index.html).


### Creating a Secondary Index

Use `Client.createIndex()` to create a secondary index. Secondary indexes are
created asynchronously, so the method returns before the secondary index
propagates to the cluster. A `Job` instance is returned in the callback
function that can be used to check the status of the index creation job.

The following example creates a numeric index *idx_foo_bar_baz* and waits for
index creation to complete. The index is in namespace _foo_ within set _bar_
and bin _baz_:

```js
var options = {
    ns: 'foo',
    set: 'bar',
    bin: 'baz',
    index: 'idx_foo_bar_baz',
    datatype: Aerospike.indexDataType.NUMERIC
}
client.createIndex(options, function (error, job) {
  if (error) {
    // error creating index
  }
  job.waitUntilDone(function (error) {
    console.log('Index was created successfully.')
  })
})
```

Secondary indexes can only be created once on the server as a combination of namespace, set, and bin name with either integer or string data types. For example, if you define a secondary index to index bin _x_ that contains integer values, then only records containing bins named _x_ with integer values are indexed. Other records with a bin named _x_ that contain non-integer values are not indexed.


When an index management call is made to any node in the Aerospike Server cluster, the information automatically propagates to the remaining nodes.

### Removing a Secondary Index

To remove a secondary index using `client.removeIndex()`:

```js
client.indexRemove('foo', 'idx_foo_bar_baz', function (error) {
})
```
