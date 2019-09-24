---
title: CDT Sub-Context Eval
description: Applying List and Map operations on deeply nested CDTs.
---

The CDT sub-context call allows [list](/docs/guide/cdt-list.html) and [map](/docs/guide/cdt-map.html) operations to target nested lists and maps respectively.

{{#note}}
This functionality was added in Aerospike version 4.6.0.
{{/note}}

## API

`subcontext_eval(context, cdt-op)`


### Context Parameter

Context Type   | |
---------------- |
BY_LIST_INDEX	 |
BY_LIST_RANK	 |
BY_LIST_VALUE	 |
BY_MAP_INDEX	 |
BY_MAP_RANK	 	 |
BY_MAP_KEY		 |
BY_MAP_VALUE	 |

The context parameter is a list of element pairs targetting a nested list or map. Each pair of values are in the form `[context-type, context-value]` specifying how to drill down to the nested element an operation is to be performed upon.

#### Drilling Down

Consider the following list:

    [0, 1, [2, [3, 4], 5, 6], 7, [8, 9] ]

We can operate on the list with value `[3, 4]` by using a context list `[BY_LIST_INDEX, 2, BY_LIST_INDEX, 1]`.

* The first pair selects the 3rd element (index 2 with 0 origin) of the top-level list, resulting in the context `[2, [3, 4], 5, 6]]`.
* The 2nd pair selects index 1 from the context resulting from the first pair.

An empty context is not allowed.

### Op Parameter

Any op from the top-level [list](/docs/guide/cdt-list-ops.html) and [map](/docs/guide/cdt-map-ops.html) APIs.

#### Examples

bin = `[0, 1, [2, [3, 4], 5, 6], 7, [8, 9] ]`

    subcontext_eval([BY_LIST_INDEX, 2, BY_LIST_INDEX, 1], list.append(100))
    >>> [0, 1, [2, [3, 4, 100], 5, 6], 7, [8, 9] ]

    subcontext_eval([BY_LIST_INDEX, 0, BY_LIST_INDEX, 1], list.append(100)) # append to non-list
    >>> Error                                                               # Invalid Context

    subcontext_eval([BY_LIST_INDEX, -1], list.append(100) )
    >>> [0, 1, [2, [3, 4, 100], 5, 6], 7, [8, 9, 100] ]

bin = `[ [0, 'a'], [1, 'b'], [3, 'd'], [2, 'c'] ]`

	subcontext_eval([BY_LIST_RANK, 0], list.append(10))
    >>> [ [0, 'a', 10], [1, 'b'], [3, 'd'], [2, 'c'] ]

    subcontext_eval([BY_LIST_RANK, -1], list.append(999))
    >>> [ [0, 'a', 10], [1, 'b'], [3, 'd', 999], [2, 'c'] ]

##### Wildcards

Search by value with [wildcard](/docs/guide/cdt-ordering.html#wildcard) token.

bin = `[ [0, 'a'], [1, 'b'], [3, 'd'], [2, 'c'] ]`

	subcontext_eval([BY_LIST_VALUE, [1, *] ], list.append(12345))
    >>> [ [0, 'a'], [1, 'b', 12345], [3, 'd'], [2, 'c'] ]

	subcontext_eval([BY_LIST_VALUE, [2, *] ], list.append( {0: 1} )) # append a map to list
    >>> [ [0, 'a'], [1, 'b', 12345], [3, 'd'], [2, 'c', {0: 1} ] ]

	subcontext_eval([BY_LIST_VALUE, [2, *] ], map.add(0, 2)) # map op on non-map
    >>> Error                                                # Invalid Context

	subcontext_eval([BY_LIST_VALUE, [2, *], BY_LIST_INDEX, -1], map.add(0, [1]))
    >>> [ [0, 'a'], [1, 'b', 12345], [3, 'd'], [2, 'c', {0: [1]} ] ]

	subcontext_eval([BY_LIST_VALUE, [2, *], BY_LIST_INDEX, -1], map.add(1, -8))
    >>> [ [0, 'a'], [1, 'b', 12345], [3, 'd'], [2, 'c', {0: [1], 1: -8} ] ]
