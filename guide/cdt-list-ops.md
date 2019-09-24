---
title: List Operations
description: Use Aerospike list operations to manipulate lists on the Aerospike server.
---

Aerospike [List](/docs/guide/cdt-list.html) operations for modifying or querying list bins. The following are server names and may differ from names implemented in your client language. See your client documentation for equivalent operations.<a name="resultType"></a>

Operation													| Op Type	| <th colspan="2" style="text-align:center;">Supported Result Types</th> | | |
-----------------------------------------------------------	| ---------	| :-------:	| :-------:	| :-----------:	| :-----------:	|
**Set Type Op**												| 			| **Index**	| **Rank **	| **IndexRange**| **RankRange**	|
[`set_type(t)`](#setType)									| bin		| <td colspan="2" style="text-align:center;">N/A</td> | |
**Modify Ops**												|			| **Index**	| **Rank **	| **IndexRange**| **RankRange**	|
[`append(v)`](#append)											| single	| <td colspan="2" style="text-align:center;">N/A</td> | |
[`insert(i,v)`](#insert)									| single	| <td colspan="2" style="text-align:center;">N/A</td> | |
[`set(i,v)`](#set)											| single	| <td colspan="2" style="text-align:center;">N/A</td> | |
[`append_items(m)`](#appendItems)								| multi		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`insert_items(i,m)`](#insertItems)							| multi		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`increment(i,dv)`](#incr)									| single	| <td colspan="2" style="text-align:center;">N/A</td> | |
[`sort()`](#sort)											| bin		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`clear()`](#clear)											| bin		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`remove_by_index(i)`](#removeByIndex)						| single	| ✓			| ✓	  		| ✓				| Ordered-only	|
[`remove_by_rank(r)`](#removeByRank)						| single	| ✓			| ✓	  		| Ordered-only	| ✓				|
[`remove_by_index_range(i,x)`](#removeByIndexRange)			| multi		| ✓			| ✓	  		| ✓				| Ordered-only	|
[`remove_by_value_interval(v0,v1)`](#removeByValueInterval)	| multi		| ✓			| ✓	  		| Ordered-only	| ✓				|
[`remove_all_by_value(v)`](#removeAllByValue)				| multi		| ✓			| ✓	  		| Ordered-only	| ✓				|
[`remove_by_rank_range(r,x)`](#removeByRankRange)			| multi		| ✓			| ✓	  		| Ordered-only	| ✓				|
[`remove_by_value_rel_rank_range(v,r,x)`](#removeByValueRelRankRange)| multi	| ✓			| ✓	  		| Ordered-only	| ✓				|
[`remove_all_by_value_list(m)`](#removeAllByValueList)		| multi		| ✓			| ✓	  		| ✗				| ✗				|
**Read Ops**												|			| **Index**	| **Rank **	| **IndexRange**| **RankRange**	|
[`size()`](#size)											| bin		| <td colspan="2" style="text-align:center;">N/A</td> | |
[`get_by_index(i)`](#getByIndex)							| single	| ✓			| ✓	  		| ✓				| Ordered-only	|
[`get_by_rank(i)`](#getByRank)								| single	| ✓			| ✓	  		| Ordered-only	| ✓				|
[`get_by_index_range(i,x)`](#getByIndexRange)				| multi		| ✓			| ✓	  		| ✓				| Ordered-only	|
[`get_by_value_interval(v0,v1)`](#getByValueInterval)		| multi		| ✓			| ✓	  		| Ordered-only	| ✓				|
[`get_all_by_value(v)`](#getAllByValue)						| multi		| ✓			| ✓	  		| Ordered-only	| ✓				|
[`get_by_rank_range(r,x)`](#getByRankRange)					| multi		| ✓			| ✓	  		| Ordered-only	| ✓				|
[`get_by_value_rel_rank_range(v,r,x)`](#getByValueRelRankRange)	| multi		| ✓			| ✓	  		| Ordered-only	| ✓				|
[`get_all_by_value_list(m)`](#getAllByValueList)			| multi		| ✓			| ✓	  		| ✗				| ✗				|

`None`, `Count`, and `Value` result types are always available for operations with a `opFlags` parameter.

`RevIndex` and `RevRank` are available whenever [Index and Rank](/docs/guide/cdt-list.html#list-terminology) are available respectively.

#### Op Type

| Op Type	| Description |
| ---------	| ----------- |
| bin		| Affects whole bin.										|
| single	| Affects 1 list entry, returns single result.				|
| multi		| Affects multiple list entries, returns a list or list of results. |

### Performance
For the operational complexity analysis of list operations, refer to the [List Performance](/docs/guide/cdt-map-performance.html) documentation.

## Details

### Modify Operations

* <a name="setType"></a>`set_type(type)`

Return: Nothing

| Type			|
| ------------- |
| UNORDERED		|
| ORDERED		|

* <a name="append"></a>`append(value[, createType, modifyFlag])`

Return: Element count after operation

Add an element v to the list.
Append v to end of list for UNORDERED lists.
Create list bin with `type=createType` if bin did not exist.

#### Modify Flags

| Flag				| Description |
| -----------------	| ----------- |
| MODIFY_DEFAULT	| Default behavior			  |
| ADD_UNIQUE		| Add elements that do not already exist in list		|
| INSERT_BOUNDED	| Do not insert past index N, where N is element count |
| NO_FAIL			| No-op instead of fail if policy violation occured, such as ADD_UNIQUE or INSERT_BOUNDED |
| DO_PARTIAL		| When used with NO_FAIL, add elements that did not violate policy |

* <a name="appendItems"></a>`append_items(items[, createType, modifyFlag])`

Return: Element count after operation

Add all entries in `items` parameter to the list.
Create list bin with `type=createType` if bin did not exist.

* <a name="insert"></a>`insert(index, value[, modifyFlag]`

Return: Element count after operation

Insert `value` at `index`. Invalid operation for ORDERED lists.

* <a name="insert"></a>`insert_items(index, items[, modifyFlag]`

Return: Element count after operation

Insert `items` at `index`. Invalid operation for ORDERED lists.

* <a name="incr"></a>`increment(index[, delta-value, createType, modifyFlag])`

Return: New value after increment

Only works for `integer` or `float` value types.
Create list bin with `type=createType` if bin did not exist.
`delta-value` defaults to 1(integer) or 1.0(float).
#### Type Interaction between `value` and `delta-value` 

| Value-type			| Delta: `integer`						| Delta: `float`							|
| --------------------- | ------------------------------------- | ----------------------------------------- |
| list Entry: `integer`	| Add normally.							| Truncate to nearest `integer` and add.	|
| list Entry: `float`	| Convert `integer` to `float` and add.	| Add normally.								|

* <a name="clear"></a>`clear()`

Return: Nothing

Remove all elements from list. List type stays the same.

* <a name="removeByIndex"></a>`remove_by_index(opFlags, index)`

Return: Single result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove element at `index`.

* <a name="removeByRank"></a>`remove_by_rank(opFlags, rank)`

Return: Single result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove the ith smallest valued element where `i == rank`.

* <a name="removeByIndexRange"></a>`remove_by_index_range(opFlags, index[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove element(s) where for all index `i`: `i >= index` and `i < index + count`.
Omitting `count` select element(s) where  `i >= index`.

* <a name="removeByValueInterval"></a>`remove_by_value_interval(opFlags, valueStart[, valueStop])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove all element(s) where `value >= valueStart` and `value < valueStop`.
Omitting `valueStop` select element(s) where `value >= valueStart`.

See [interval comparison](/docs/guide/cdt-ordering.html#intervals).

* <a name="removeAllByValue"></a>`remove_all_by_value(opFlags, value)`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove all element(s) equal to `value`.

* <a name="removeByRankRange"></a>`remove_by_rank_range(opFlags, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove all element(s) where `r` = rankof(element) and `r >= rank` and `r < rank + count`.
Omitting `count` select element(s) where `r >= rank`.

* <a name="removeByValueRelRankRange"></a>`remove_by_value_rel_rank_range(opFlags, value, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove all element(s) where `origin = rank(value)`, `r` = rankof(element) and `r >= origin + rank` and `r < origin + rank + count`.
Omitting `count` select element(s) where `r >= origin + rank`.

{{#note}}
Relative operations added since version 4.3
{{/note}}

* <a name="removeAllByValueList"></a>`remove_all_by_value_list(opFlags, valueList)`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Remove all element(s) where `list.value ∈ valueList`.

### Read Operations

* <a name="size"></a>`size()`

Return: Element count

* <a name="getByIndex"></a>`get_by_index(opFlags, index)`

Return: Single result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get element at `index`.

* <a name="getByRank"></a>`get_by_rank(opFlags, rank)`

Return: Single result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get the ith smallest valued element where `i == rank`.

* <a name="getByIndexRange"></a>`get_by_index_range(opFlags, index[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get element(s) where for index `i`: `i >= index` and `i < index + count`.
Omitting `count` select element(s) where  `i >= index`.

* <a name="getByValueInterval"></a>`get_by_value_interval(opFlags, valueStart[, valueStop])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get all element(s) where `value >= valueStart` and `value < valueStop`.
Omitting `valueStop` select element(s) where `value >= valueStart`.

See [interval comparison](/docs/guide/cdt-ordering.html#intervals).

* <a name="getAllByValue"></a>`get_all_by_value(opFlags, value)`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get all element(s) equal to `value`.

* <a name="getByRankRange"></a>`get_by_rank_range(opFlags, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get all element(s) where `r` = rankof(element) and `r >= rank` and `r < rank + count`.
Omitting `count` select element(s) where `r >= rank`.

* <a name="getByValueRelRankRange"></a>`get_by_value_rel_rank_range(opFlags, value, rank[, count])`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get all element(s) where `origin = rank(value)`, `r` = rankof(element) and `r >= origin + rank` and `r < origin + rank + count`.
Omitting `count` select element(s) where `r >= origin + rank`.

* <a name="getAllByValueList"></a>`get_all_by_value_list(opFlags, valueList)`

Return: Multi result, see [resultType](/docs/guide/cdt-list.html#resultType) table

Get all element(s) where `list.value ∈ valueList`.
