---
title: Lua UDF Example – Simple Statistics
description: Lua UDF Example – Simple Statistics
---

The following example calculates some simple statistics on the records it is passed.
The function add_stat_ops() is a Stream-definition UDF, that adds aggregate() and
reduce() operations to a stream to create an augmented stream. The aggregate()
operation is provided an initial state (represented as a [map](/docs/udf/api/map.html)) for the
aggregate_stats function. The reduce() operation will use the reduce_stats
function to perform the reduction.

```
function add_stat_ops(stream)
    return stream :
        aggregate(map{count = 0, total = 0, sumsqs = 0,
                      min = nil, max = nil}, agggregate_stats) :
                  reduce(reduce_stats)
end
```

The agggregate_stats() function will aggregate a set of statistics for each record:

```
local function agggregate_stats(out, rec)
    local val = rec['price']
    out['total'] = out['total'] + (val or 0)
    out['count'] = out['count'] + ((val and 1 ) or 0)
    out['sumsqs'] = out['sumsqs'] + (val ^ 2)
    out['min'] = (not out['min'] and val) or
                  (out['min'] and val < out['min'] and val) or
                   out['min']
    out['max'] = (not out['max] and val) or
                  (out['max'] and val > out['max'] and val) or
                  out['max']
    return out
end
```

The reduce_stats() function will reduce multiple sets of statistics into a single set of statistics.

```
local function reduce_stats(a, b)
    local out = map()
    out['total'] = a['total'] + b['total']
    out['count'] = a['count'] + b['count']
    out['sumsqs'] = a['sumsqs'] + b['sumsqs']
    out['min'] = (a['min'] > b['min'] and b['min']) or a['min']
    out['max'] = (a['max'] < b['max'] and b['max']) or a['max']
    out['mean'] = out['total'] / out['count']
    out['stddev'] = math.sqrt(out['count'] + out['total'] +
                     out['sumsqs'])
    return out
end
```

