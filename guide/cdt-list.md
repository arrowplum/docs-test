---
title: Lists
description: Use Aerospike list operations to manipulate lists on the Aerospike server.
---

Aerospike [Lists](https://en.wikipedia.org/wiki/List_%28abstract_data_type%29) can store [scalar datatypes](/docs/guide/data-types.html#basic-types) or contain another list or [map](/docs/guide/cdt-map.html).
Aerospike [list operations](/docs/guide/cdt-list-ops.html) are optimal for manipulating lists directly on the Aerospike server. For example, to add or remove a list item without reading and then replacing the whole bin value. See [List Examples](/docs/guide/cdt-list-examples.html).

## List Terminology

For the following List, `[ 1, 4, 7, 3, 9, 26, 11 ]`:

 * The **index** is the (0 origin) position of an element in the list.
 * The **value** at index 2 is 7. The value at index -2 is 26.
 * The **rank** is the value order of the elements in the list. The lowest value element has rank 0.
 * The value of the element with rank 2 is 4. The value of the element with rank -2 is 11.

## Element Ordering

Element access can be specified through index, rank (value ordering) or element value.

    Unordered [ 1, 4, 7, 3, 9, 26, 11 ]
    Ordered   [ 1, 3, 4, 7, 9, 11, 26 ]

### Ordered Lists
 * An ordered list maintains rank order, and sorts itself every time new elements are added.
 * In an ordered list, elements of mixed data types are first [ordered by type, then by value](/docs/guide/cdt-ordering.html).

### Unordered Lists
 * An unordered list maintains insertion order, as long as new elements are appended.
 * In an unordered list, getting elements by rank will obey [the same](/docs/guide/cdt-ordering.html) (just-in-time) ordering.


## List API

Using the Aerospike client API, an application can retrieve the full list, or operate on individual elements. Atomic [list  operations](/docs/guide/cdt-list-ops.html) work on top level elements with the following API, and on nested elements with an additional **[subcontext-eval API](/docs/guide/cdt-context.html)**.

The following operations all have [language-specific client implementations](/docs/guide/cdt-list.html#language-specific-client-operations).

* [`set_type()`](/docs/guide/cdt-list-ops.html#setType)
* [`append()`](/docs/guide/cdt-list-ops.html#append), [`insert()`](/docs/guide/cdt-list-ops.html#insert), [`insert_items()`](/docs/guide/cdt-list-ops.html#insertItems), [`append_items()`](/docs/guide/cdt-list-ops.html#appendItems)
* [`set()`](/docs/guide/cdt-list-ops.html#set)
* [`increment()`](/docs/guide/cdt-list-ops.html#incr), [`sort()`](/docs/guide/cdt-list-ops.html#sort)
* [`clear()`](/docs/guide/cdt-list-ops.html#clear), [`size()`](/docs/guide/cdt-list-ops.html#size)
* [`remove_by_index()`](/docs/guide/cdt-list-ops.html#removeByIndex), [`remove_by_index_range()`](/docs/guide/cdt-list-ops.html#removeByIndexRange)
* [`remove_by_rank()`](/docs/guide/cdt-list-ops.html#removeByRank), [`remove_by_rank_range()`](/docs/guide/cdt-list-ops.html#removeByRankRange), [`remove_by_value_rel_rank_range()`](/docs/guide/cdt-list-ops.html#removeByValueRelRankRange)
* [`remove_by_value()`](/docs/guide/cdt-list-ops.html#removeByValue), [`remove_by_value_interval()`](/docs/guide/cdt-list-ops.html#removeByValueInterval), [`remove_all_by_value()`](/docs/guide/cdt-list-ops.html#removeAllByValue), [`remove_all_by_value_list()`](/docs/guide/cdt-list-ops.html#removeAllByValueList)
* [`get_by_index()`](/docs/guide/cdt-list-ops.html#getByIndex), [`get_by_index_range()`](/docs/guide/cdt-list-ops.html#getByIndexRange)
* [`get_by_rank()`](/docs/guide/cdt-list-ops.html#getByRank), [`get_by_rank_range()`](/docs/guide/cdt-list-ops.html#getByRankRange)
* [`get_by_value()`](/docs/guide/cdt-list-ops.html#getByValue), [`get_by_value_interval()`](/docs/guide/cdt-list-ops.html#getByValueInterval), [`get_all_by_value()`](/docs/guide/cdt-list-ops.html#getAllByValue), [`get_by_value_rel_rank_range()`](/docs/guide/cdt-list-ops.html#getByValueRelRankRange), [`get_all_by_value_list()`](/docs/guide/cdt-list-ops.html#getAllByValueList)

{{#note}}
Operations on nested list/map elements added in Aerospike version 4.6.
{{/note}}
{{#note}}
Relative operations added in Aerospike version 4.3.
{{/note}}

#### Superceded operations

These operations are superceded by the `remove_by_*()` and `get_by_*()` operations and may possibly be deprecated in a future release.

- `get()`, `get_range()`, `get_range_from()`
- `pop()`, `pop_range()`
- `remove()`, `remove_range()`
- `trim()`

### List Operation Flag
<a name="opFlag"></a>

`get_*` or `remove_*` operations have an `opFlags` parameter to control what they return back and to modify their search criteria.

OpFlag	    | Description 	  | |
-----------	| ----------------- |
INVERTED	| Inverts the search criteria provided in the op parameters.	|

- Remove the 10 elements with the largest values: `remove_by_rank_range(-10, 10, NONE)`
- Keep only the 10 elements with the largest values: `remove_by_rank_range(-10, 10, INVERTED)`

{{#note}}
Since Aerospike version 3.16.0
{{/note}}

### List Result Types
<a name="resultType"></a>
`get_*` and `remove_*` operations have a `resultType` parameter to control what they return back.

Result Type | Description 	  | |
-----------	| ----------------- |
None		| No results, useful for faster remove operations due to not constructing a reply.	|
Index		| Position order: 0 = smallest position, -1 = largest position. 	|
RevIndex	| Reverse index order: 0 = largest position.						|
Rank		| Value order: 0 = smallest value, -1 = largest value.				|
RevRank		| Reverse value order: 0 = largest value.							|
Count		| Return number of items matching criteria.							|
Value		| Value for single item operations, list of values for multi-ops.	|

- Keep only the top 10 values, return count: `remove_by_rank_range(-10, 10, INVERTED | COUNT)`

{{#note}}
Since Aerospike version 3.16.0, `resultType` folded into `opFlags`.
{{/note}}

## Development Guidelines and Tips

- A list bin is created when a list value is written to a bin, or by using list `append`, `insert`, or `set` operations.
- Insertion at either end of the list requires no element walk and is generally fast.
- If insertions and deletions are mostly at the end of the list, operations are&ndash;on average&ndash;fast.
- For data-in-memory, internal structures are optimized for random-access reads.
- The in-memory lazy index is filled on reads so that subsequent reads on the same list benefit from previous reads, unless a list modification occurred.
- If the namespace saves to disk storage, disk writes may create a bottleneck.
 For example, if there are not enough nodes or devices or if key use is not evenly distributed. This is especially true for list bins nearing maximum record size.

## Performance

For the operational complexity analysis of list operations see [List Performance](/docs/guide/cdt-map-performance.html).

## Known Limitations

- Lists are bound by the maximum record size. For data on SSD, the maximum record size is limited by the `write-block-size` configuration parameter.
- List operations are not supported by Lua UDFs.

## Language Specific Client Operations

Language-specific list operation examples:

* [Java](https://github.com/aerospike/aerospike-client-java/tree/master/client/src/com/aerospike/client/cdt)
* [C](https://github.com/aerospike/aerospike-client-c/blob/master/examples/basic_examples/list/src/main/example.c)
* [C#](https://github.com/aerospike/aerospike-client-csharp/tree/master/Framework/AerospikeDemo)
* [Go](https://github.com/aerospike/aerospike-client-go/tree/master/examples)
* [Python](https://github.com/aerospike/aerospike-client-python/tree/master/examples)
* [Node.js](https://github.com/aerospike/aerospike-client-nodejs/tree/master/examples)
* [Ruby](https://github.com/aerospike/aerospike-client-ruby/tree/master/examples)
* [PHP](https://github.com/aerospike/aerospike-client-php/tree/master/examples)
* [Rust](https://github.com/aerospike/aerospike-client-rust/)
