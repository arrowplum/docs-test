
Operation 	 		 	  		| ResultType   	  | Unordered (1)	| K / KV-Ordered (2) | K-Ordered (3)	| KV-Ordered (4)	| |
-------------------------------	| --------------- | --------------- | --------------- | ------------------- | ----------------- |
**Set Type Ops**				| 				  |					|**With No Index**|**With Offset Index**|**With Full Index**|
`set_type(t)` on unordered		|				  | 1				| 1				  | 1					| 1					|
`set_type(t)` on k_ordered		|				  | N log N			| 1				  | 1					| 1					|
`set_type(t)` on kv_ordered		|				  | N log N			| N log N		  | N log N				| 1					|
**Modify Ops**					|				  |					|**With No Index**|**With Offset Index**|**With Full Index**|
`add(k,v)`						|				  | N				| N				  | log N				| log N				|
`add_items(m)`					|				  | N·M				| N·M			  | M log N				| M log N			|
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
`remove_by_index_range(i,x)`	| Base			  | (L+M)log N + N	| N + M			  | M					| M	  				|
								| +Rank			  | +N log N		| +N log N		  | +N log N			| +log N			|
`remove_by_value_interval(v0,v1)`<br>`remove_all_by_value(v)`| Base| N + M | N + M	  | N + M				| log N + M			|
								| +Index		  | +N + M			| +M			  | +M					| +M				|
`remove_by_rank_range(r,x)`		| Base			  | (R+M)log N + N	| (R+M)log N + N  | (R+M) log N			| M log M			|
								| +Index		  | +N log N		| +M			  | +M					| +M				|
`remove_all_by_key_list(m)`		| Base			  | M log M + N·M	| M log M + N·M	  | M log N				| M log N			|
								| +Index		  | +N·M			| +M			  | +M					| +M				|
`remove_all_by_value_list(m)`	| Base			  | M log M + N·M	| M log M + N·M	  | M log M + N·M		| M log N			|
**Read Ops**					|				  |					|**With No Index**|**With Offset Index**|**With Full Index**|
`size()` 						|				  | 1				| 1				  | 1					| 1	  				|
`get_by_key(k)`					| <td colspan="4" style="text-align:center;">See `remove_by_key` above.</td>					|
`get_by_index(i)`				| <td colspan="4" style="text-align:center;">See `remove_by_index` above.</td>					|
`get_by_rank(i)`				| <td colspan="4" style="text-align:center;">See `remove_by_index` above.</td>					|
`get_by_key_interval(v0,v1)`	| Base			  | N				| N				  | log N + M			| log N + M			|
								| +Rank			  | +N log N		| +N log N		  | +N log N			| +log N			|
`get_by_index_range(i,x)`		| <td colspan="4" style="text-align:center;">See `remove_by_index_range` above.</td>			|
`get_by_value_interval(v0,v1)`<br>`get_all_by_value(v)` | <td colspan="4" style="text-align:center;">See `remove_by_value_interval` above.</td>	|
`get_by_rank_range(r,x)`		| Base			  | (R+M)log N + N	| (R+M)log N + N  | (R+M) log N			| M					|
								| +Index		  | +N log N		| +M			  | +M					| +M				|
**Mode Modifiers**				|				  |					|**With No Index**|**With Offset Index**|**With Full Index**|
Not-in-memory On-Disk			| <td colspan="4" style="text-align:center;">+D + W</td>										|
Data-in-memory On-Disk			| <td colspan="4" style="text-align:center;">+W</td>											|
