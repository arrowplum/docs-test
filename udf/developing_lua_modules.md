---
title: Developing UDF Modules
description: Leverage the fast, powerful, lightweight embedable scripting language Lua, 
assets: /docs/udf/assets
---

### Modules Written in Lua

A Lua [`module`](https://www.lua.org/manual/5.1/manual.html#5.3) is a collection of variables and functions contained in a single file. Modules can be imported and used by other Lua modules, including UDFs. A module name must not conflict with other Lua objects pre-registered by the Aerospike server. The following module names should be avoided:

* aerospike
* bytes
* geojson
* iterator
* list
* map
* record
* stream

#### Creating a Module
In the following example, we define a local table `exports` in the file _mymodule.lua_. This module will be
populated with the functions to be exported.

```lua
local exports = {}

function exports.hello()
  return "Hello "
end

function exports.world()
  return "World!"
end

return exports
```

#### Registering a Module

Lua Modules must be registered with the Aerospike Server.

To install your modules, you may

* Use the [aql](/docs/tools/aql/index.html) tool
* Use any of the Aerospike Clients

To register the module using aql:

```sql
aql> register module 'mymodule.lua'
OK, 1 module added.

aql> show modules
+-----------------+--------------------------------------------+-------+
| filename        | hash                                       | type  |
+-----------------+--------------------------------------------+-------+
| "mymodule.lua"  | "4e4dfd2ac120e161f69d1dfbab14eec157d0eaaf" | "LUA" |
+-----------------+--------------------------------------------+-------+
```

Another module can now `require` _mymodule.lua_ into a local variable and use
it.

#### Example: A Hello World UDF

In a file _example.lua_

```lua
local mm = require('mymodule')

function helloworld(rec)
  return mm.hello() .. mm.world()
end
```

In aql

```sql
aql> register module 'example.lua'
OK, 1 module added.

aql> show modules
+-----------------+--------------------------------------------+-------+
| filename        | hash                                       | type  |
+-----------------+--------------------------------------------+-------+
| "example.lua"   | "c42bf3f4a6f8f727efcfb884c97ee894764f1dc7" | "LUA" |
| "mymodule.lua"  | "4e4dfd2ac120e161f69d1dfbab14eec157d0eaaf" | "LUA" |
+-----------------+--------------------------------------------+-------+
2 rows in set (0.002 secs)

aql> insert into test.foo (PK, x) values ('1', 24)
OK, 1 record affected.

aql> execute example.helloworld() on test.foo where PK='1'
+----------------+
| helloworld     |
+----------------+
| "Hello World!" |
+----------------+
1 row in set (0.001 secs)
```

### Using C functions in UDF Modules

Aerospike UDFs written in Lua can call C functions from shared objects. For more
information read about the [C API for Lua](https://www.lua.org/manual/5.1/manual.html#3).

**Note:** Make sure you compile the module to be loaded into Aerospike using the correct version of Lua.

#### Example: Compiling and Registering a Shared Object

In this example the file _power.c_ contains sample C code:

```c
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>

static int go(lua_State * L) {
  int rtrn = lua_tonumber(L, -1);   /* Get the single number arg */
  lua_pushnumber(L,rtrn*rtrn);      /* square it and push the return */

  return 1;
}

static const struct luaL_reg golib [] = {
  {"go", go},
  {NULL, NULL}
};

int luaopen_power(lua_State * L) {
  luaL_openlib(L, "go", golib, 0);
  return 1;
}
```

The Lua UDF module _use.lua_ requires and calls the _power.so_ shared object

```lua
function thepower(rec, basenum)
  local power = require("power")
  local rtn = power.go(basenum);
  --info(rtn)
  return rtn
end
```

##### Get the Lua Source
Download the [Lua 5.1.4 library](https://sourceforge.net/projects/luabinaries/files/5.1.4/Linux%20Libraries/) to allow for the necessary Lua code to be linked.

##### Compile
```
gcc -fPIC -o power.so -shared power.c -I /usr/include/
```

##### Register and Execute

In aql:

```sql
aql> register module 'power.so'
OK, 1 module added.

aql> register module 'use.lua'
OK, 1 module added.

aql> show modules
+-----------------+--------------------------------------------+-------+
| filename        | hash                                       | type  |
+-----------------+--------------------------------------------+-------+
| "example.lua"   | "c42bf3f4a6f8f727efcfb884c97ee894764f1dc7" | "LUA" |
| "power.so"      | "99703e01482f065c62f0d0c55ba6b1f3214e3601" | "LUA" |
| "use.lua"       | "5093704498b333e57d7dd033b529c4b4e0d99edb" | "LUA" |
| "mymodule.lua"  | "4e4dfd2ac120e161f69d1dfbab14eec157d0eaaf" | "LUA" |
+-----------------+--------------------------------------------+-------+
4 rows in set (0.002 secs)

aql> execute use.thepower(4) on test.foo where PK='1'
+----------+
| thepower |
+----------+
| 16       |
+----------+
1 row in set (0.001 secs)
```

