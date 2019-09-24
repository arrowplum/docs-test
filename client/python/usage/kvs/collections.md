---
title: Collection Operations
description: Use the Aerospike Python client to perform operations on collections in the Aerospike database.
---

Use the Aerospike Python client to perform operations on [Lists](/docs/guide/cdt-list.html) in the Aerospike database.

One of the supported data types of the Aerospike database are size bounded lists that can reside in a single bin. The operations below allow manipulation of the list on the server &mdash; for example to add or remove an item &mdash; without the need to read and replace the whole bin value the list is stored in.

### Operations Available on Lists

The following Client operations are supported on [Lists](/docs/guide/cdt-list.html) values:
 
Operation | Description
--- | ---
list_append | Adds an element to the end of the list.
list_extend | Adds a list of elements to the end of the list.
list_insert | Inserts an element at the specified index.
list_insert_items | Inserts a list of elements at the specified index.
list_pop | Removes and returns the list element at the specified index.
list_pop_range | Removes and returns the list elements at the specified range.
list_remove | Removes the list element at the specified index.
list_remove_range | Removes the list elements at the specified range.
list_clear | Removes all elements from the list.
list_set | Sets a list element at the specified index to a new value.
list_trim | Removes all list elements not within the specified range.
list_get | Returns the list element at the specified index.
list_get_range | Returns the list of elements at the specified range.
list_size | Returns the element count of the list.

### Example code

This examples illustrates writing a list to the Aerospike database and then appending an item to the list.

```python
key = ('test', 'demo', 1)
rec = {'coutry': 'India', 'city': ['Pune', 'Dehli']}

client.put(key, rec)

client.list_append(key, 'city', 'Mumbai')


```
