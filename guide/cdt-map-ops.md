---
title: Map Operations
description: Use Aerospike map operations to manipulate maps on the Aerospike server.
---

Aerospike [Map](/docs/guide/cdt-map.html) operations for modifying or querying map bins. The following are server names and may differ from names implemented in your client language. See your client documentation for equivalent operations.

Operation													| Op Type	| <th colspan="2" style="text-align:center;">Supported Result Types</th> | | |
-----------------------------------------------------------	| ---------	| :-------:	| :-------:	| :-----------:	| :-----------:	|
**Set Type Op**												| 			| **Index**	| **Rank **	| **IndexRange**| **RankRange**	|
[`set_type(t)`](#setType)									| bin		| <td colspan="2" style="text-align:center;">N/A</td> | |
**Modify Ops**												|			| **Index**	| **Rank **	| **IndexRange**| **RankRange**	|
[`add(k,v)`](#add)											| single	| <td colspan="2" style="text-align:center;">N/A</td> | |
[`add_items(m)`](#addItems)									| multi		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`increment(k,dv)`](#incr)									| single	| <td colspan="2" style="text-align:center;">N/A</td> | |
[`decrement(k,dv)`](#incr)									| single	| <td colspan="2" style="text-align:center;">N/A</td> | |
[`clear()`](#clear)											| bin		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`remove_by_key(k)`](#removeByKey)							| single	| ✓			| ✓	  		| ✓				| ✗				|
[`remove_by_index(i)`](#removeByIndex)						| single	| ✓			| ✓	  		| ✓				| ✗				|
[`remove_by_rank(r)`](#removeByRank)						| single	| ✓			| ✓	  		| ✗				| ✓				|
[`remove_by_key_interval(v0,v1)`](#removeByKeyInterval)		| multi		| ✓			| ✓	  		| ✓				| ✗				|
[`remove_by_index_range(i,x)`](#removeByIndexRange)			| multi		| ✓			| ✓	  		| ✓				| ✗				|
[`remove_by_value_interval(v0,v1)`](#removeByValueInterval)	| multi		| ✓			| ✓	  		| ✗				| ✓				|
[`remove_all_by_value(v)`](#removeAllByValue)				| multi		| ✓			| ✓	  		| ✗				| ✓				|
[`remove_by_rank_range(r,x)`](#removeByRankRange)			| multi		| ✓			| ✓	  		| ✗				| ✓				|
[`remove_by_key_rel_index_range(r,x)`](#removeByKeyRelIndexRange)| multi	| ✓			| ✓	  		| ✓				| ✗				|
[`remove_by_value_rel_rank_range(r,x)`](#removeByValueRelRankRange)| multi	| ✓			| ✓	  		| ✗				| ✓				|
[`remove_all_by_key_list(m)`](#removeAllByKeyList)			| multi		| ✓			| ✗	  		| ✗				| ✗				|
[`remove_all_by_value_list(m)`](#removeAllByValueList)		| multi		| ✓			| ✓	  		| ✗				| ✗				|
**Read Ops**												|			| **Index**	| **Rank **	| **IndexRange**| **RankRange**	|
[`size()`](#size)											| bin		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`get_by_key(k)`](#getByKey)								| single	| ✓			| ✓	  		| ✓				| ✗				|
[`get_by_index(i)`](#getByIndex)							| single	| ✓			| ✓	  		| ✓				| ✗				|
[`get_by_rank(i)`](#getByRank)								| single	| ✓			| ✓	  		| ✗				| ✓				|
[`get_by_key_interval(v0,v1)`](#getByKeyInterval)			| multi		| ✓			| ✓	  		| ✓				| ✗				|
[`get_by_index_range(i,x)`](#getByIndexRange)				| multi		| ✓			| ✓	  		| ✓				| ✗				|
[`get_by_value_interval(v0,v1)`](#getByValueInterval)		| multi		| ✓			| ✓	  		| ✗				| ✓				|
[`get_all_by_value(v)`](#getAllByValue)						| multi		| ✓			| ✓	  		| ✗				| ✓				|
[`get_by_rank_range(r,x)`](#getByRankRange)					| multi		| ✓			| ✓	  		| ✗				| ✓				|
[`get_by_key_rel_index_range(r,x)`](#getByKeyRelIndexRange)	| multi		| ✓			| ✓	  		| ✓				| ✗				|
[`get_by_value_rel_rank_range(r,x)`](#getByValueRelRankRange)	| multi		| ✓			| ✓	  		| ✗				| ✓				|
[`get_all_by_key_list(m)`](#getAllByKeyList)				| multi		| ✓			| ✗	  		| ✗				| ✗				|
[`get_all_by_value_list(m)`](#getAllByValueList)			| multi		| ✓			| ✓	  		| ✗				| ✗				|

`None`, `Count`, `Key`, `Value` and `KeyValue` result types are always available for operations with a `resultType` parameter.

`RevIndex` and `RevRank` are available whenever [Index and Rank](/docs/guide/cdt-map.html#map-terminology) are available respectively.

#### Op Type

| Op Type	| Description |
| ---------	| ----------- |
| bin		| Affects whole bin.										|
| single	| Affects 1 map entry, returns single result.				|
| multi		| Affects multiple map entries, returns a list or map of results. |

### Performance
For the operational complexity analysis of map operations, refer to the [Map Performance](/docs/guide/cdt-map-performance.html) documentation.

## Details

### Modify Operations

* <a name="setType"></a>`set_type(type)`

Return: Nothing

| Type			|
| ------------- |
| UNORDERED		|
| K_ORDERED		|
| KV_ORDERED	|

* <a name="add"></a>`add(key, value[, createType])`

Return: Element count after add

Add a `{key: value}` pair to the map.
Create map bin with `type=createType` if bin did not exist.

#### Entry Overwrite/Create Logic

| Op Name	| Policy		| Key Exist | Key Not Exist |
| ---------	| ------------- | --------- | ------------- |
| add		| CREATE_ONLY	| no-op		| create new	|
| put		| UPDATE		| overwrite	| create new	|
| replace	| UPDATE_ONLY	| overwrite	| no-op			|

Some clients may use a map policy rather than different op name to control overwrite/create logic.

* <a name="addItems"></a>`add_items(items[, createType])`

Return: Element count after operation

Add all entries in `items` parameter to the map.
Create map bin with `type=createType` if bin did not exist.

* <a name="put"></a>`put(key, value[, createType, modifyFlag])`

Return: Element count after operation

#### Modify Flags
{{#note}}
Since version 4.3
{{/note}}

| Flag				| Description |
| -----------------	| ----------- |
| MODIFY_DEFAULT	| Default behavior			  |
| NO_OVERWRITE		| Replacing existing elements not allowed		|
| NO_CREATE			| Creating new elements not allowed				|
| NO_FAIL			| No-op instead of fail if policy violation occured, such as NO_OVERWRITE or NO_CREATE |
| DO_PARTIAL		| When used with NO_FAIL, add elements that did not violate policy |

* <a name="put"></a>`put_items(key, value[, createType, modifyFlag])`

Return: Element count after operation

* <a name="incr"></a>`increment(createType, key, delta-value)`, `decrement(createType, key, delta-value)`

Return: New value after increment/decrement

Only works for `integer` or `float` value types in the `{key: value}` pair.
Create map bin with `type=createType` if bin did not exist.

#### Type Interaction between `value` and `delta-value` 

| Value-type			| Delta: `integer`						| Delta: `float`							|
| --------------------- | ------------------------------------- | ----------------------------------------- |
| Map Entry: `integer`	| Add normally.							| Truncate to nearest `integer` and add.	|
| Map Entry: `float`	| Convert `integer` to `float` and add.	| Add normally.								|

Use subtract in place of add for `decrement` operation.

* <a name="clear"></a>`clear()`

Return: Nothing

Clears the map. Map type stays the same.

* <a name="removeByKey"></a>`remove_by_key(resultType, key)`

Return: Single result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove entry `{key: value}` where `map.key == key`.

* <a name="removeByIndex"></a>`remove_by_index(resultType, index)`

Return: Single result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove `{key: value}` entry where `map.key` is the ith smallest key where `i == index`.

* <a name="removeByRank"></a>`remove_by_rank(resultType, rank)`

Return: Single result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove `{key: value}` entry where `map.value` is the ith smallest value where `i == rank`.

* <a name="removeByKeyInterval"></a>`remove_by_key_interval(resultType, keyStart[, keyStop])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `map.key >= keyStart` and `map.key < keyStop`.
Omitting `keyStop` select element(s) where `map.key >= keyStart`.

See [interval comparison](/docs/guide/cdt-ordering.html#intervals).

* <a name="removeByIndexRange"></a>`remove_by_index_range(resultType, index[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `k` = indexof(map.key) and `k >= index` and `k < index + count`.
Omitting `count` select element(s) where `k >= origin + index`.

* <a name="removeByValueInterval"></a>`remove_by_value_interval(resultType, valueStart[, valueStop])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `map.value >= valueStart` and `map.value < valueStop`.
Omitting `valueStop` select element(s) where `map.value >= valueStart`.

See [interval comparison](/docs/guide/cdt-ordering.html#intervals).

* <a name="removeAllByValue"></a>`remove_all_by_value(resultType, value)`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `map.value == value`.

* <a name="removeByRankRange"></a>`remove_by_rank_range(resultType, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `r` = rankof(map.value) and `r >= rank` and `r < rank + count`.
Omitting `count` select element(s) where `r >= rank`.

* <a name="removeByKeyRelIndexRange"></a>`remove_by_key_rel_index_range(resultType, key, index[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `origin` = index(key), `k` = indexof(map.key) and `k >= origin + index` and `k < origin + index + count`.
Omitting `count` select element(s) where `k >= origin + index`.

* <a name="removeByValueRelRankRange"></a>`remove_by_value_rel_rank_range(resultType, value, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `origin` = rank(value), `r` = rankof(map.value) and `r >= origin + rank` and `r < origin + rank + count`.
Omitting `count` select element(s) where `r >= origin + rank`.

{{#note}}
Relative operations added since version 4.3
{{/note}}

* <a name="removeAllByKeyList"></a>`remove_all_by_key_list(resultType, keyList)`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `map.key ∈ keyList`.

* <a name="removeAllByValueList"></a>`remove_all_by_value_list(resultType, valueList)`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Remove all `{key: value}` pairs where `map.value ∈ valueList`.
{{#info}}
INDEX result supported since version 3.16.0
{{/info}}

### Read Operations

* <a name="size"></a>`size()`

Return: Element count

* <a name="getByKey"></a>`get_by_key(resultType, key)`

Return: Single result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get entry `{key: value}` where `map.key == key`.

* <a name="getByIndex"></a>`get_by_index(resultType, index)`

Return: Single result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get `{key: value}` entry where `map.key` is the ith smallest key where `i == index`.

* <a name="getByRank"></a>`get_by_rank(resultType, rank)`

Return: Single result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get `{key: value}` entry where `map.value` is the ith smallest value where `i == rank`.

* <a name="getByKeyInterval"></a>`get_by_key_interval(resultType, keyStart[, keyStop])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `map.key >= keyStart` and `map.key < keyStop`.
Omitting `keyStop` select element(s) where `map.key >= keyStart`.

See [interval comparison](/docs/guide/cdt-ordering.html#intervals).

* <a name="getByIndexRange"></a>`get_by_index_range(resultType, index[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `k` = indexof(map.key) and `k >= index` and `k < index + count`.
Omitting `count` select element(s) where `k >= origin + index`.

* <a name="getByValueInterval"></a>`get_by_value_interval(resultType, valueStart[, valueStop])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `map.value >= valueStart` and `map.value < valueStop`.
Omitting `valueStop` select element(s) where `map.value >= valueStart`.

See [interval comparison](/docs/guide/cdt-ordering.html#intervals).

* <a name="getAllByValue"></a>`get_all_by_value(resultType, value)`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `map.value == value`.

* <a name="getByRankRange"></a>`get_by_rank_range(resultType, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `r` = rankof(map.value) and `r >= rank` and `r < rank + count`.
Omitting `count` select element(s) where `r >= rank`.

* <a name="getByKeyRelIndexRange"></a>`get_by_key_rel_index_range(resultType, key, index[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `origin` = index(key), `k` = indexof(map.key) and `k >= origin + index` and `k < origin + index + count`.
Omitting `count` select element(s) where `k >= origin + index`.

* <a name="getByValueRelRankRange"></a>`get_by_value_rel_rank_range(resultType, value, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `origin` = rank(value), `r` = rankof(map.value) and `r >= origin + rank` and `r < origin + rank + count`.
Omitting `count` select element(s) where `r >= origin + rank`.

* <a name="getAllByKeyList"></a>`get_all_by_key_list(resultType, keyList)`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `map.key ∈ keyList`.

* <a name="getAllByValueList"></a>`get_all_by_value_list(resultType, valueList)`

Return: Multi result, see [resultType](/docs/guide/cdt-map.html#resultType) table

Get all `{key: value}` pairs where `map.value ∈ valueList`.
