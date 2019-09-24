---
title: List Performance
description: Computational complexity of list operations
---

List performance is usually dictated by the element access.

### Table of Element Walk Complexities (Worst Case)

- `D`&mdash;Cost of load from disk
- `W`&mdash;Cost of write to disk
- `N`&mdash;element count of list
- `M`&mdash;element count of parameter, range or interval
- `R`&mdash;for rank r, R = min(r, N - r - 1)
- `L`&mdash;for index i, L = min(i, N - i - 1)
- `C`&mdash;Cost of memcpy on packed list
- `n`&mdash;The total byte size of list elements.

Every modify op has a +`C` for copy on write for allowing rollback on failure.

Operation 	 		 	  		| ResultType   	  | Unordered (1)	| Ordered (2)     | Ordered (3)	      | |
-------------------------------	| --------------- | --------------- | --------------- | ------------------- |
**Set Type Ops**				| 				  |					|**With No Index**|**With Offset Index**|
`set_type(t)` to unordered		|				  | 1				| 1				  | 1					|
`set_type(t)` to ordered		|				  | N log N			| 1				  | 1					|
**Modify Ops**					|				  |					|**With No Index**|**With Offset Index**|
`add(v)`<br>`append(v)`<br>`insert(0, v)`|		  | 1				| log N + N		  | log N				|
UNIQUE add/append				|				  | N				| log N + N		  | log N				|
`add_items(m)`<br>`append_items(m)`<br>`insert_items(0, m)`| | M	| (M+N)log M + N  |	(M+N)log M + M 		|
UNIQUE add/append items		    |				  | N*M				| (M+N)log M + N  |	(M+N)log M + M 		|
`insert(i > 0, v)`				|				  | N				| -				  | -					|
`insert_items(i > 0, m)`		|				  | M + N			| -				  | -					|
`increment(i,dv)`				|				  | N 				| -	  			  | -					|
`sort()`						|				  | N log N			| 1				  | 1					|
UNIQUE sort						|				  | N log N			| N				  | N					|
`clear()`						|				  | 1				| 1				  | 1					|
`remove_by_index(i)`			| Base			  | N				| N				  | 1					|
`remove_by_value(v)`			| Base			  | N				| log N	+ N		  | log N				|
`remove_by_rank(r)`				| Base			  | R log N + N		| N				  | 1					|
`remove_by_index_range(i,x)`	| Base			  | N + M			| N + M			  | M					|
								| +Rank			  | +L*N			|				  |						|
`remove_by_value_interval(v0,v1)`<br>`remove_all_by_value(v)`| Base| N + M | log N + N| log N + M			|
`remove_by_rank_range(r,x)`		| Base			  | R log N + N		| N + M			  | M					|
`remove_by_value_rel_rank_range(r,x)`| Base		  | R log N + N		| N + M	 + log N  | M + log N			|
`remove_all_by_value_list(m)`	| Base			  | (M+N)log M + N	| (M+N)log M + N  | (M+N)log M			|
**Read Ops**					|				  |					|**With No Index**|**With Offset Index**|
`size()` 						|				  | 1				| 1				  | 1					|
`get_by_index(i)`				| <td colspan="3" style="text-align:center;">See `remove_by_index` above.</td>	|
`get_by_value(v)`				| <td colspan="3" style="text-align:center;">See `remove_by_value` above.</td>	|
`get_by_rank(i)`				| <td colspan="3" style="text-align:center;">See `remove_by_index` above.</td>	|
`get_by_index_range(i,x)`		| <td colspan="3" style="text-align:center;">See `remove_by_index_range` above.</td>			|
`get_by_value_interval(v0,v1)`<br>`get_all_by_value(v)` | <td colspan="3" style="text-align:center;">See `remove_by_value_interval` above.</td>	|
`get_by_rank_range(r,x)`		| <td colspan="3" style="text-align:center;">See `remove_by_rank_range` above.</td>				|
`get_by_value_rel_rank_range(r,x)`		| <td colspan="3" style="text-align:center;">See `remove_by_value_rel_rank_range` above.</td>				|
`get_all_by_value_list(m)`		| <td colspan="3" style="text-align:center;">See `remove_all_by_value_list` above.</td>			|
**Mode Modifiers**				|				  |					|				  |						|
Not-in-memory On-Disk			| <td colspan="3" style="text-align:center;">+D + W</td>					|
Data-in-memory On-Disk			| <td colspan="3" style="text-align:center;">+W</td>						|

- Lists in in-memory namespaces with persistence have a lookup index, which allows a random access average of `O(1)`. The worst case is `O(N)` because it is a lazy index.
- Namespaces with data on SSD have no lookup index and incur an additional `O(n)` hit while reading whole records from disk.

### Persistence and Replication

For namespaces with data on SSD, an additional `O(n)` *memcpy* performance cost is incurred. **This is negligible** compared to element walk `O(N)`.

The replication cost is `O(n)`, because full records are replicated and persisted on replicas.

