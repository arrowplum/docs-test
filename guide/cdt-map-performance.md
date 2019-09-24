---
title: Map Performance
description: Computational complexity of map operations
---

### Table of Element Walk Complexities (Worst Case)

- `D`&mdash;Cost of load from disk
- `W`&mdash;Cost of write to disk
- `N`&mdash;element count of map
- `M`&mdash;element count of parameter, range or interval
- `R`&mdash;for rank r, R = min(r, N - r - 1)
- `L`&mdash;for index i, L = min(i, N - i - 1)
- `C`&mdash;Cost of memcpy on packed map

Every modify op has a +`C` for copy on write for allowing rollback on failure.

Operation 	 		 	  		| ResultType   	  | Unordered (1)	| K / KV-Ordered (2) | K-Ordered (3)	| KV-Ordered (4)	| |
-------------------------------	| --------------- | --------------- | --------------- | ------------------- | ----------------- |
**Set Type Ops**				| 				  |					|**With No Index**|**With Offset Index**|**With Full Index**|
`set_type(t)` on unordered		|				  | 1				| 1				  | 1					| 1					|
`set_type(t)` on k_ordered		|				  | N log N			| 1				  | 1					| 1					|
`set_type(t)` on kv_ordered		|				  | N log N			| N log N		  | N log N				| 1					|
**Modify Ops**					|				  |					|**With No Index**|**With Offset Index**|**With Full Index**|
`add(k,v)`						|				  | N				| N				  | log N				| log N				|
`add_items(m)`					|				  | (N+M)log M + N	| M(log M + log N) + N| M(log M + log N) + N | M(log M + log N)	+ N |
`increment(k,dv)`<br>`decrement(k,dv)`|			  | N 				| N	  			  | log N 				| log N				|
`clear()`						|				  | 1				| 1				  | 1					| 1	  				|
`remove_by_key(k)`				| Base			  | N				| N				  | log N				| log N				|
								| +Index		  | +N				| +N			  | +N					| +log N			|
`remove_by_index(i)`			| Base			  | L log N	+ N		| N				  | 1					| 1	  				|
								| +Rank			  | +N				| +N			  | +N					| +log N			|
`remove_by_rank(r)`				| Base			  | R log N + N		| R log N + N	  | R log N				| 1	  				|
								| +Index		  | +N				| +1			  | +1					| +1				|
`remove_by_key_interval(v0,v1)`	| Base			  | N + M			| N	+ M			  | log N + M			| log N + M			|
								| +Rank			  | +N log N		| +N log N		  | +N log N			| +log N			|
`remove_by_index_range(i,x)`	| Base			  | L log N + N		| N + M			  | M					| M	  				|
								| +Rank			  | +N log N		| +N log N		  | +N log N			| +log N			|
`remove_by_value_interval(v0,v1)`<br>`remove_all_by_value(v)`| Base| N + M | N + M	  | N + M				| log N + M			|
								| +Index		  | +N + M			| +M			  | +M					| +M				|
`remove_by_rank_range(r,x)`		| Base			  | R log N + N		| R log N + N	  | R log N				| M					|
								| +Index		  | +N log N		| +M			  | +M					| +M				|
`remove_all_by_key_list(m)`		| Base			  | (M+N)logM + N	| M log N + N	  | M log N				| M log N			|
`remove_all_by_value_list(m)`	| Base			  | (M+N)logM + N	| (M+N)log M + N  | (M+N)log M			| M log N			|
**Read Ops**					|				  |					|**With No Index**|**With Offset Index**|**With Full Index**|
`size()` 						|				  | 1				| 1				  | 1					| 1	  				|
`get_by_key(k)`					| <td colspan="4" style="text-align:center;">See `remove_by_key` above.</td>					|
`get_by_index(i)`				| <td colspan="4" style="text-align:center;">See `remove_by_index` above.</td>					|
`get_by_rank(i)`				| <td colspan="4" style="text-align:center;">See `remove_by_index` above.</td>					|
`get_by_key_interval(v0,v1)`	| Base			  | N				| N				  | log N + M			| log N + M			|
								| +Rank			  | +N log N		| +N log N		  | +N log N			| +log N			|
`get_by_index_range(i,x)`		| <td colspan="4" style="text-align:center;">See `remove_by_index_range` above.</td>			|
`get_by_value_interval(v0,v1)`<br>`get_all_by_value(v)` | <td colspan="4" style="text-align:center;">See `remove_by_value_interval` above.</td>	|
`get_by_rank_range(r,x)`		| <td colspan="4" style="text-align:center;">See `remove_by_rank_range` above.</td>				|
`get_all_by_key_list(m)`		| <td colspan="4" style="text-align:center;">See `remove_all_by_key_list` above.</td>			|
`get_all_by_value_list(m)`		| <td colspan="4" style="text-align:center;">See `remove_all_by_value_list` above.</td>			|
**Mode Modifiers**				|				  |					|				  |						|					|
Not-in-memory On-Disk			| <td colspan="4" style="text-align:center;">+D + W</td>										|
Data-in-memory On-Disk			| <td colspan="4" style="text-align:center;">+W</td>											|

[Complexity table for versions < 3.16.0](/docs/guide/cdt-map-complex0.html)

### Performance Analysis Example

Consider a storage model where the map key is a name and the map value is the score associated with the name.

`map: {name1: score1, name2: score2, ...}`

Let's say you want to optimize performance for getting the names with the top 10 scores. The operation for getting the top 10 would be `get_by_rank_range`.

`get_by_rank_range(rank=-10, count=10, opFlags=key);`

#### Data-in-memory

From the complexities table, one can see choosing KV-Ordered (4) for `get_by_rank_range` will result in O(M) performance. Since this is data-in-memory, KV-Ordered has full map indexing. Also, because we're getting top 10, the performance is O(10) or constant time.

There's is an additional +`C` cost due to the copy on write nature of the database, and +`W` if there is disk backing for this namespace.

Performance: O(1) + C [+ W]

#### Data-on-SSD

Namespaces with data on SSD have no map indexing, so the choice is only between columns (1) and (2). For `get_by_rank_range`, they are the same: O((R + M)log N + N).

Performance: O((R + M) log N + N) + D + W

#### Return Type Extra Cost

Looking at `get_by_rank_range` again, if the result type was index, there would be extra performance cost as stated on the row below `get_by_rank_range` with `+Index` under the `Result Type` column.

For a K-ordered data-in-memory map for example, the base performance of `get_by_rank_range` is:
O((R + M) log N)

For `get_by_rank_range(rank, count, opFlags=index)`:
O((R + M) log N + M)

