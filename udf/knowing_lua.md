---
title: Getting Started With Lua UDFs
description: Learn how you can get started using User-Defined Functions using Lua and C. 
assets: /docs/udf/assets
---

### Introduction
Currently, User-Defined Functions are written in Lua.
Lua is a powerful, fast, lightweight, embeddable scripting language.
To learn more about Lua, see [About Lua](http://www.lua.org/about.html).

If this is your first foray into Lua programming, then we suggest the following material:

* [Programming in Lua, Second Edition (Lua 5.1)](http://www.amazon.com/exec/obidos/ASIN/8590379825/lua-pilindex-20)
* [Lua 5.1 Reference Manual](http://www.lua.org/manual/5.1/)
* [Lua Tutorials](http://lua-users.org/wiki/TutorialDirectory)
* [Lua Unofficial FAQ (uFAQ)](http://www.luafaq.org/)

### Lua Version

Aerospike currently supports Lua 5.1.4.

### Lua Restrictions

Aerospike supports the full Lua programming language, with a few exceptions.

#### Globals are restricted

* Global variables are not allowed.
* Global functions can only be called by Aerospike Server, and cannot be called by other Lua functions. 
* To call a custom Lua function from another Lua function, the called function must be declared as a "local" function.  For example, external function **sum()** can call local function **add()**, provided it is defined as local.

```
	local function add(a,b)
  		return a + b
	end
	 
	function sum(a,b)
  		return add(a,b)
	end
```

#### Restricted modules and functions

* coroutines - Lua functions can call each other provided they are "forward declared" before their actual use.  In this example, we forward-declare fun\_B() and then we can use it inside the body of fun\_B(). We don't have to forward-declare fun_A() in this example because it is declared first.

```
	local fun_B
	
	local fun_A( foo )
		fun_B( foo.bar )
	end
	
	local fun_B( bar )
		fun_A( bar.foo )
	end
```

* debug module – Not enabled due to not being able to support the debugging features in Lua.
* os.exit() – Not enabled because it didn't make sense for a Lua script to cause the database to process to exit.
