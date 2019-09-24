---
title: Aggregate Records
description: Use the Aerospike PHP client to aggregate query results.
---

Use the Aerospike PHP client to aggregate query results of a [secondary index query](/docs/client/php/usage/udf/query.html).

For developers familiar with aggregate functions in SQL, the following statement counts rows from the database:

```sql
SELECT COUNT(*)
FROM test.users
WHERE age >= 20 AND age <= 29
GROUP BY first_name
```

This statement uses the `COUNT()` aggregate function gets the count of users grouped by the column *first_name* from a query of *test.users*, where the value of the column *age* is in the twenties.

### Aggregating Query Results with a Stream UDF

Use the Aerospike PHP client [aggregate()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_query.md) function to combine the query using a [Stream UDF](/docs/guide/stream_udf.html).

1. Construct the following query:

  ```php
// assuming test.users has a bin first_name, show the first name distribution
// for users in their twenties
$where = Aerospike::predicateBetween("age", 20, 29);
$status = $db->aggregate("test", "users", $where, "mymod", "group_count", ["first_name"], $names);
if ($status == Aerospike::OK) {
    var_dump($names);
} else {
    echo "An error occured while running the AGGREGATE [{$db->errorno()}] ".$db->error();
}
  ```

1. [Register the UDF](/docs/client/php/usage/udf/manage.html) with the database. 

  This example uses the *mymod* UDF module to the define `group_count()` [Stream UDF](/docs/guide/stream_udf.html):

```lua
local function having_ge_threshold(bin_having, ge_threshold)
    return function(rec)
        debug("group_count::thresh_filter: %s >  %s ?", tostring(rec[bin_having]), tostring(ge_threshold))
        if rec[bin_having] < ge_threshold then
            return false
        end
        return true
    end
end

local function count(group_by_bin)
  return function(group, rec)
    if rec[group_by_bin] then
      local bin_name = rec[group_by_bin]
      group[bin_name] = (group[bin_name] or 0) + 1
      debug("group_count::count: bin %s has value %s which has the count of %s", tostring(bin_name), tostring(group[bin_name]))
    end
    return group
  end
end

local function add_values(val1, val2)
  return val1 + val2
end

local function reduce_groups(a, b)
  return map.merge(a, b, add_values)
end

function group_count(stream, group_by_bin, bin_having, ge_threshold)
  if bin_having and ge_threshold then
    local myfilter = having_ge_threshold(bin_having, ge_threshold)
    return stream : filter(myfilter) : aggregate(map{}, count(group_by_bin)) : reduce(reduce_groups)
  else
    return stream : aggregate(map{}, count(group_by_bin)) : reduce(reduce_groups)
  end
end
```

The following are the expected Stream UDF results:

```
array(5) {
  ["Claudio"]=>
  int(1)
  ["Michael"]=>
  int(3)
  ["Jennifer"]=>
  int(2)
  ["Jessica"]=>
  int(3)
  ["Jonathan"]=>
  int(3)
}
```

