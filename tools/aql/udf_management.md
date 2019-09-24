---
title: aql – UDF Management
description: Learn how aql can be used for UDF management, to register, list describe or drop a UDF module.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: aql
    url: /docs/v3/aql.html
---

### Registering a Module

The following is the command to register a UDF Module:

```sql
REGISTER MODULE '<filepath>'
```

This command uploads the file at the given path `<filepath>`. The name of the file will be the module's name.

The following is an example of uploading a UDF module at `~/tmp/my_udf.lua`

```sql
aql> register module '~/tmp/my_udf.lua'
```
When you register a module, the module is automatically copied to all nodes in the cluster. If you can run the module from one cluster node but not another, verify the following:

- The cluster is healthy
- The user that runs Aerospike also owns owns `/opt/aerospike/smd/*`

### Listing Modules

The following is the command to list all registered modules:

```sql
SHOW MODULES
```

The following is an example of a listing:

```sql
aql> show modules
+---------------------------+-------+------------------------+
| module                    | type  | hash                   |
+---------------------------+-------+------------------------+
| "example1.lua"            | "lua" | "033671e05067888fce09" |
| "example2.lua"            | "lua" | "07b42082cca8e73a96b2" |
+---------------------------+-------+------------------------+
2 rows in set (0.000 secs)
```

The module information includes the module name and type (currently, only Lua is supported) and the hash value of the file.  Most users will not find the hash value useful, but some may use it to verify the version or instance of a UDF on the server.

### Describe a Module

The following is the command to describe a registered module:

```sql
DESC MODULE <module>
```

This will describe the UDF module named `<module>`.

The describe command shows the UDF. In the gen field (basically, a hash value), the module type (Lua)  and the text of the entire UDF in the content field. The output is most legible in raw format ('set output raw').

The following is an example:

```sql
aql> set output raw
OUTPUT = RAW
aql> desc module example2.lua
*************************** 1. row ***************************
content: "
local function bin(name)
    local function x(rec)
        return rec[name]
    end
    return x
end

local function even(a)
    return a % 2 == 0
end

local function add(a,b)
    return a + b
end

local function multiplier(factor)
    local function x(a)
        return a * factor
    end
    return x
end

function foo(rec)
    return bin("a")(rec)
end

function a(stream)
    return stream : map(bin("a"))
end

function even_a(stream)
    return stream : map(bin("a")) : filter(even)
end

function sum_even_a(stream)
    return stream : map(bin("a")) : filter(even) : reduce(add)
end

function sum_even_a_x(stream, factor)
    return stream : map(bin("a")) : filter(even) : reduce(add) : map(multiplier(factor))
end"
gen: "13fgxKWlQDHukDDp0/fsrjxvcMA="
type: "LUA"

[127.0.0.1:3000] 1 row in set (0.011 secs)
```

### Dropping a Module

The following is the command to drop (remove) a registered module from the cluster:

```sql
REMOVE MODULE <module>
```

This will drop (remove) the UDF module named `<module>`.

The following is an example:

```sql
aql> remove module example2.lua
```
