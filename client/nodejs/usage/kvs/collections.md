---
title: Collection Operations
description: Use the Aerospike Node.js client to perform operations on collections in the Aerospike database.
---

Use the Aerospike Node.js client to perform operations on
[Lists](/docs/guide/cdt-list.html) and [Maps](/docs/guide/cdt-map.html) in the
Aerospike database.

In addition to the basic data types such as integers or strings, the Aerospike
database supports several Complex Data Types (CDT). Two of these CDTs are
Lists and Maps. The operations below allow manipulation of list and maps on the
server &mdash; for example to add or remove an item &mdash; without the need to
read and replace the whole bin value the list/map is stored in.

### Operations Available on Lists

The following operations are supported on [List](/docs/guide/cdt-list.html) values:
 
Operation     | Description
------------- | ------------------
append        | Adds an element to the end of the list.
appendItems   | Adds a list of elements to the end of the list.
insert        | Inserts an element at the specified index.
insertItems   | Inserts a list of elements at the specified index.
pop           | Removes and returns the list element at the specified index.
popRange      | Removes and returns the list elements at the specified range.
remove        | Removes the list element at the specified index.
removeRange   | Removes the list elements at the specified range.
clear         | Removes all elements from the list.
set           | Sets a list element at the specified index to a new value.
trim          | Removes all list elements not within the specified range.
get           | Returns the list element at the specified index.
getRange      | Returns the list of elements at the specified range.
size          | Returns the element count of the list.

### Examples

These examples illustrate common operations on collections.

#### Tracking Page Views

This example application tracks the page views of a website. The key is the URL for a page. The record contains the following bins: 

- `last-updated` &mdash;  (integer) The POSIX time this record was last updated.
- `views` &mdash; (integer) The number of page view entries. 
- `addr` &mdash; (list) A list of IP address strings.
- `user` &mdash; (list) A list of user ID strings.
- `time` &mdash; (list) A list of timestamps.

The `addr`, `user`, and `time` bins are [Lists](/docs/guide/cdt-list.html) of scalar values (Strings, Integers). Their entries are in sync in that a single page-view value is at the same index of each bin.

```js
let ts = new Date().getTime()
let addr = req.connection.remoteAddress
let user = req.session.user

// Get the operators object from aerospike module to perform multiple 
// operations on a record.
let op = Aerospike.operations
let lists = Aerospike.lists

// Array of operations to be performed on the record
var ops = [
  op.write('last-updated', ts),
  op.incr('views', 1),
  op.read('views'),
  lists.append('addr', addr),
  lists.append('user', user),
  lists.append('time', ts)
]

// Key of the record on which the operations have to be performed
let key = new Aerospike.Key('app', 'pages', uid)

// operate on the record
client.operate(key, ops, function (error, record) {
  if (error) {
    console.log(error.message)
  } else {
    let bins = record.bins
    let views = bins['views'] // post-increment value
  }
})
```

### Operations Available on Maps

The following operations are supported on [Map](/docs/guide/cdt-map.html) values:
 
Operation          | Description
------------------ | ----------------
setPolicy          | Sets the map policy (sort order)
put                | Updates a map entry to a new value or creates a new map entry.
putItems           | Updates/creates multiple map entries.
increment          | Increments the numeric value of an existing map entry.
decrement          | Decrements the numeric value of an existing map entry.
clear              | Removes all elements from the map.
removeByKey        | Removes a single key from a map; optionally returns the removed entry.
removeByKeyList    | Removes on or more keys from a map; optionally returns the removed entries.
removeByKeyRange   | Removes a range of keys from a map; optionally returns the removed entries.
removeByValue      | Removes a single map entry identified by value; optionally returns the removed entry.
removeByValueList  | Removes one or more map entries identified by value; optionally returns the removed entries.
removeByValueRange | Removes a range of map entries identified by value; optionally returns the removed entries.
removeByIndex      | Removes the map entry identified by index; optionally returns the removed entry.
removeByIndexRange | Removes a range of map entries identified by index; optionally returns the removed entries.
removeByRank       | Removes the map entry identified by rank; optionally returns the removed entry.
removeByRankRange  | Removes a range of map entries identified by rank; optionally returns the removed entries.
size               | Returns the number of entries in the map.
getByKey           | Returns a single map entry identified by key.
getByKeyRange      | Returns a range of map entries identified by key.
getByValue         | Returns the map entries identified by value.
getByValueRange    | Returns a range of map entries identified by value.
getByIndex         | Returns the map entry identified by index.
getByIndexRange    | Returns a range of map entries identified by index.
getByRank          | Returns a map entry identified by rank.
getByRankRange | Returns a range of map entries identified by rank.
