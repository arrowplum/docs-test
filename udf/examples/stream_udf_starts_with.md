---
title: String Starts With Stream UDF Example
description: Lua stream UDF example
---

The recs\_start\_with() stream UDF filters for strings in the _bin_ that have a specific _prefix_.

The local filter function starts\_with() is a closure that takes the pair of
arguments, and returns the required signature of a filter function. The filter
function can now check whether a string value inside _bin_ starts with the
prefix.


```lua
local function map_record(rec)
  local ret = map()
  for i, bin_name in ipairs(record.bin_names(rec)) do
    ret[bin_name] = rec[bin_name]
  end
  return ret
end

local function starts_with(bin, prefix)
  return function(rec)
    if rec[bin] then
      local s = rec[bin]
      if type(s) == 'string' then
        return s:sub(1, #prefix) == prefix
      end
    end
    return false
  end
end

function recs_start_with(s, bin, prefix)
  return s : filter(starts_with(bin, prefix)) : map(map_record)
end
```

This stream UDF has a mapper only (no reducer), which is used to cast records that
pass the filter into a map of bin-name, bin-value pairs. The record itself is not
a valid return type for a UDF.

