---
title: CDT Element Ordering and Comparison
description: Element ordering for Aerospike list and map.
---

## Type Order (Ascending)

Elements with different types are ordered based on their type.

1. NIL
2. BOOLEAN
3. INTEGER
4. STRING
5. LIST
6. MAP
7. BYTES
8. DOUBLE
9. GEOJSON
10. INF

### NIL
The lowest valued type. NIL is a singleton.

### BOOLEAN
`False < True`

### INTEGER
Ordered by integer value.

### STRING
Order by each byte in the string.

`"aa" < "b"`

Strings are assumed to have UTF-8 encoding.

### LIST
Order by:

1. Each element starting from index 0 `[1, 2] < [1, 3]`
2. Element count `[1, 2] < [1, 2, 1]`

{{#warn}}
There is a known issue where Map/List comparisons may be incorrect for maps/lists of different lengths. If planning to utilize comparisons please update to Aerospike Server 4.3.1.x or higher.
{{/warn}}

### MAP
Order by:

1. Element count
2. Each key in order stored

{{#warn}}
There is a known issue where Map/List comparisons may be incorrect for maps/lists of different lengths. If planning to utilize comparisons please update to Aerospike Server 4.3.1.x or higher.
{{/warn}}

### BYTES
Order by each byte in the string.

### DOUBLE
Ordered by float value.

### INF
Introduced in Aerospike Server v4.3.1.x

The highest valued type. INF is a singleton.

Not a storage type. Storing INF in a bin list or map has undefined behavior.

{{#note}}
Introduced in Aerospike Server v4.3.1.x.
{{/note}}

## Comparison

### Wildcard
The singleton WILDCARD(*) type can be passed as parameters in certain operations. WILDCARD is not a storage type.

Bin: `[ [1, 1], [1, 2], [1, 3], [2, 1], [2, 2], [2, 3] ]`

`list_get_all_by_value([1, *]) -> [ [1, 1], [1, 2], [1, 3] ]`

{{#note}}
Introduced in Aerospike Server v4.3.1.x.
{{/note}}

### Intervals
Intervals are inclusive-exclusive by default: start <= elements < end

Bin: `[ [1, 1], [1, 2], [2, 1], [2, 2], [3, 1] ]`

`list_get_by_value_interval(start=[1, NIL], end=[2, NIL]) -> [ [1, 1], [1, 2] ]`

#### INF
Using INF, we can get an inclusive-inclusive interval when using 2nd level lists.

`list_get_by_value_interval(start=[1, NIL], end=[2, INF]) -> [ [1, 1], [1, 2], [2, 1], [2, 2] ]`

{{#note}}
Introduced in Aerospike Server v4.3.1.x.
{{/note}}
