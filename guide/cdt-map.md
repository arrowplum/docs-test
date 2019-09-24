---
title: Maps
description: Use Aerospike map operations to manipulate maps on the Aerospike server.
---

Aerospike [Maps](https://en.wikipedia.org/wiki/Associative_array)  can store [scalar datatypes](/docs/guide/data-types.html#basic-types) or contain another [list](/docs/guide/cdt-list.html) or map.
Aerospike [map operations](/docs/guide/cdt-map-ops.html) are optimal for manipulating key-value pairs directly on the Aerospike server. For example, to add or remove items without reading and then replacing the whole bin value. See [Map Examples](/docs/guide/cdt-map-examples.html).

## Map Terminology

For the following map, `{a:1, b:2, c:13, z:26, y:13}`

 * The **index** is the order of a particular key in the map. The index of `b` is 1. The index of `y` is -2.
 * The **value** of a is 1. The value of `z` is 26.
 * The **rank** is the [order of a particular value](/docs/guide/cdt-ordering.html) in the map.
 * If multiple copies of a value exist, the ranks of those copies are based on their key ordering.
 * The rank of `y` is 3. The rank of `z` is -1. The rank of `c` is 2.

## Element Ordering

Elements are stored based on the map's ordering type. Unordered maps have no persisted ordering, while K-ordered maps are stored based on key ordering. KV-ordered maps are also K-ordered but may have an additional value-order index depending the namespace configuration.

Element access can also be based on [ordering](/docs/guide/cdt-ordering.html) - either through index (key ordering) or rank (value ordering).

Irrespective of the intrisic storage ordering of the map, both index or rank based operations are supported.

## Map APIs

Using the Aerospike client API, an application can retrieve the full map, or operate on individual elements. Atomic [map  operations](/docs/guide/cdt-map-ops.html) work on top level elements with the following API, and on nested elements with an additional **[subcontext-eval API](/docs/guide/cdt-context.html)**.

The following operations all have [language-specific client implementations](/docs/guide/cdt-map.html#language-specific-client-operations).

* [`set_type()`](/docs/guide/cdt-map-ops.html#setType)
* [`add()`](/docs/guide/cdt-map-ops.html#add), [`add_items()`](/docs/guide/cdt-map-ops.html#addItems), [`increment()`](/docs/guide/cdt-map-ops.html#incr), [`decrement()`](/docs/guide/cdt-map-ops.html#incr), [`clear()`](/docs/guide/cdt-map-ops.html#clear)
* [`remove_by_key()`](/docs/guide/cdt-map-ops.html#removeByKey), [`remove_by_index()`](/docs/guide/cdt-map-ops.html#RemoveByIndex), [`remove_by_rank()`](/docs/guide/cdt-map-ops.html#removeByRank)
* [`remove_by_key_interval()`](/docs/guide/cdt-map-ops.html#removeByKeyInterval), [`remove_by_index_range()`](/docs/guide/cdt-map-ops.html#removeByIndexRange)
* [`remove_by_value_interval()`](/docs/guide/cdt-map-ops.html#removeByValueInterval), [`remove_by_rank_range()`](/docs/guide/cdt-map-ops.html#removeByRankRange), [`remove_all_by_value()`](/docs/guide/cdt-map-ops.html#removeAllByValue)
* [`remove_by_key_rel_index_range()`](/docs/guide/cdt-map-ops.html#removeByKeyRelIndexRange), [`remove_by_value_rel_rank_range()`](/docs/guide/cdt-map-ops.html#removeByValueRelRankRange)
* [`remove_all_by_key_list()`](/docs/guide/cdt-map-ops.html#removeAllByKeyList), [`remove_all_by_value_list()`](/docs/guide/cdt-map-ops.html#removeAllByValueList)
* [`size()`](/docs/guide/cdt-map-ops.html#size)
* [`get_by_key()`](/docs/guide/cdt-map-ops.html#getByKey), [`get_by_index()`](/docs/guide/cdt-map-ops.html#getByIndex), [`get_by_rank()`](/docs/guide/cdt-map-ops.html#getByRank)
* [`get_by_key_interval()`](/docs/guide/cdt-map-ops.html#getByKeyInterval), [`get_by_index_range()`](/docs/guide/cdt-map-ops.html#getByIndexRange)
* [`get_by_value_interval()`](/docs/guide/cdt-map-ops.html#getByValueInterval), [`get_by_rank_range()`](/docs/guide/cdt-map-ops.html#getByRankRange), [`get_all_by_value()`](/docs/guide/cdt-map-ops.html#getAllByValue)
* [`get_by_key_rel_index_range()`](/docs/guide/cdt-map-ops.html#getByKeyRelIndexRange), [`get_by_value_rel_rank_range()`](/docs/guide/cdt-map-ops.html#getByValueRelRankRange)
* [`get_all_by_key_list()`](/docs/guide/cdt-map-ops.html#getAllByKeyList), [`get_all_by_value_list()`](/docs/guide/cdt-map-ops.html#getAllByValueList)

{{#note}}
Operations on nested list/map elements added in Aerospike version 4.6.
{{/note}}
{{#note}}
Relative operations added in Aerospike version 4.3.
{{/note}}

### Map Operation Flag
<a name="opFlag"></a>

`get_*` or `remove_*` operations have an `opFlags` parameter to control what they return back and to modify their search criteria.

OpFlag	    | Description 	  | |
-----------	| ----------------- |
INVERTED	| Inverts the search criteria provided in the op parameters.	|

- Remove the 10 map elements with the largest key values: `remove_by_index_range(-10, 10, NONE)`
- Remove all but the 10 map elements with the largest key values: `remove_by_index_range(-10, 10, INVERTED)`

{{#note}}
Since Aerospike version 3.16.0
{{/note}}

#### Map Result Types
<a name="resultType"></a>
`get_*` and `remove_*` operations have a `resultType` parameter to control what they return back.

Result Type | Description 	  | |
-----------	| ----------------- |
None		| No results, useful for faster remove operations due to not constructing a reply.	|
Index		| Key order: 0 = smallest key, -1 = largest key.  	  	 	 		|
RevIndex	| Reverse key order: 0 = largest key.								|
Rank		| Value order: 0 = smallest value, -1 = largest value.				|
RevRank		| Reverse value order: 0 = largest value.							|
Count		| Return number of items matching criteria.							|
Key			| Key for single item operations and list of keys for multi-ops.	|
Value		| Value for single item operations, list of values for multi-ops.	|
KeyValue	| Key Value pairs. The exact format is client dependent.			|

- Keep only the top 10 key elements, return count: `remove_by_index_range(-10, 10, INVERTED | COUNT)`

{{#note}}
Since Aerospike version 3.16.0, `resultType` folded into `opFlags`.
{{/note}}

## Development Guidelines and Tips

- A map bin is created when a map value is written to a bin, or by using map `add`, `add_items`, or `increment` operations.
- Use `set_type()` to convert an unordered map into a K-ordered or KV-ordered map. Alternatively, use a map policy with `add` or `add_items` and the `map_type` in the policy will be used to create the map bin if it did not exist.
- All map operations work for any `map_type`. The only difference is performance as detailed in the performance table below.
- In general, when namespace is data-on-SSD, choosing column (2) **K-ordered gives the best performance** for all map operations at the cost of 4 extra bytes on disk. This is because data-on-SSD does not carry indexes around (to save space on disk) and because being K-ordered has no performance drawbacks for all map operations.
- The interval-based operations have parameters [start, stop) where the start value is inclusive and stop value is exclusive. They are generally faster than range-based operations when there is no map ordering and/or when data is not in memory (due to lack of map index meta data).
- The range-based operations have parameters (start, count) where start is an index or rank, are very fast for ordered maps within data-in-memory namespaces.
- Key-based operations are those with the suffixes `by_index`, `by_index_range` and suffixes with the word `key`. These operations generally incur additional performance penalties when returning `rank` type results. They are usually sped up by being K-ordered and having offset index meta data (available when namespace is data-in-memory).
- Value-based operations are those with the suffixes `by_rank`, `by_rank_range` and suffixes with the word `value`. These operations generally incur additional performance penalties when returning `index` type results. They are usually sped up by being KV-ordered and having full index meta data (namespace is data-in-memory).

## Performance

For the operational complexity analysis of map operations, refer to the [Map Performance](/docs/guide/cdt-map-performance.html) documentation.


## Known Limitations

- Maps are bound by the maximum record size. For data-on-SSD, the maximum record size is limited by the `write-block-size` configuration parameter.
- Map operations are not supported by Lua UDFs.

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
