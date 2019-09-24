---
title: Collection Operations
description: Use the Aerospike Ruby client to perform operations on collections in the Aerospike database.
---

Use the Aerospike Ruby client to perform operations on
[Lists](/docs/guide/cdt-list.html) and [Maps](/docs/guide/cdt-map.html) in the
Aerospike database.

In addition to the basic data types such as integers or strings, the Aerospike
database supports several Complex Data Types (CDT). Two of these CDTs are
Lists and Maps. The operations below allow manipulation of list and maps on the
server &mdash; for example to add or remove an item &mdash; without the need to
read and replace the whole bin value the list/map is stored in.

### Operations Available on Lists

The following operations are supported on [List](/docs/guide/cdt-list.html) values:
 
Operation | Description
--- | ---
append | Adds one or more elements to the end of the list.
insert | Inserts one or more elements at the specified index.
pop | Removes and returns the list element at the specified index.
pop_range | Removes and returns the list elements at the specified range.
remove | Removes the list element at the specified index.
remove_range | Removes the list elements at the specified range.
set | Sets a list element at the specified index to a new value.
trim | Removes all list elements not within the specified range.
clear | Removes all elements from the list.
size | Returns the element count of the list.
get | Returns the list element at the specified index.
get_range | Returns the list of elements at the specified range.

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

```ruby
require "aerospike"
include Aerospike

client = Client.new
ts = Time.now.to_i
addr = request.remote_ip
user = current_user.id

# Array of operations to be performed on the record
ops = [
  Operation.put(Bin.new('last-updated', ts)),
  Operation.add(Bin.new('views', 1)),
  Operation.get('views'),
  CDT::ListOperation.append("addr", addr),
  CDT::ListOperation.append("user", user),
  CDT::ListOperation.append("time", ts)
]

# Key of the record on which the operations have to be performed
key = Key.new("app", "pageviews", uid)

# operate on the record
result = client.operate(key, ops)
views = result.bins['views'] # post-increment value
```

### Operations Available on Maps

The following operations are supported on [Map](/docs/guide/cdt-map.html) values:
 
Operation | Description
--- | ---
set_policy | Sets the map policy (sort order)
put | Updates a map entry to a new value or creates a new map entry.
put_items | Updates/creates multiple map entries.
increment | Increments the numeric value of an existing map entry.
decrement | Decrements the numeric value of an existing map entry.
clear | Removes all elements from the map.
remove_keys | Removes on or more keys from a map; optionally returns the removed entries.
remove_key_range | Removes a range of keys from a map; optionally returns the removed entries.
remove_values | Removes one or more map entries identified by value; optionally returns the removed entries.
remove_value_range | Removes a range of map entries identified by value; optionally returns the removed entries.
remove_index | Removes the map entry identified by index; optionally returns the removed entry.
remove_index_range | Removes a range of map entries identified by index; optionally returns the removed entries.
remove_by_rank | Removes the map entry identified by rank; optionally returns the removed entry.
remove_by_rank_range | Removes a range of map entries identified by rank; optionally returns the removed entries.
size | Returns the number of entries in the map.
get_key | Returns the map entry identified by key.
get_key_range | Returns a range of map entries identified by key.
get_value | Returns the map entries identified by value.
get_value_range | Returns a range of map entries identified by value.
get_index | Returns the map entry identified by index.
get_index_range | Returns a range of map entries identified by index.
get_by_rank | Returns a map entry identified by rank.
get_by_rank_range | Returns a range of map entries identified by rank.
