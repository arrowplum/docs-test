---
title: map
description: The Aerospike map module introduces the Map type and functions to Lua and provides an consistent interface for maps, which is not provided by Lua tables.
---

### Usage

The map module introduces the Map type and functions to Lua. It provides a consistent interface for maps, which is not provided by Lua tables.  In stream and record UDFs the Map, which is a type supported by the database, should be used in place of Lua tables.

To create an empty Map, you can simply call the map() function.

```lua
> local m = map()
Map()
```

To create an empty Map with a specified initial capacity (the default is 32), you can call the map.create() function.

```lua
> local m = map.create(1000)
Map()
```

If you want to initialize the Map with values, then you can provide an associative Lua Table:

```lua
> local m = map { a = 1, b = 2, c = 3 }
Map( "a" => 1, "b" => 2, "c" => 3 )
```

You can access the Map much like you do with Lua Tables:

```lua
> m['a']
1
   
> m.b
2
 
> m.b = 7
> m.b
7
```

Note that because assigning a nil value in a Lua table deletes the entry from the table, it is not possible
to initialize a Map containing nil values via the map() function.

A map must only contain values of the following types:

- integer
- string
- [bytes](/docs/udf/api/bytes.html)
- [list](/docs/udf/api/list.html)
- [map](/docs/udf/api/map.html)

Placing other Lua types - for example, functions or tables - will result in run-time errors.

### Functions

-------------------------------------------------------------------------------
#### map()

Creates a new Map. [Its initial capacity will be 32.  If the map will contain many more entries than this, use map.create() for better performance.]

```lua
function map(t: Table?): Map
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`t` – (optional) Lua table containing the initial values.
    </ul>
  </dd>
  <dt>Returns
  <dd>The intialized `Map`
</dl>

Examples:

Create an empty Map:

```lua
> local m = map()
Map()
```

Create a new Map and initialize it via an associative Lua Table:

```lua
> local m = map { a = 1, b = 2, c = 3 }
Map( "a" => 1, "b" => 2, "c" => 3 )
```


-------------------------------------------------------------------------------
#### map.create()

Creates a new Map, with a specified initial capacity.  For greatest efficiency, specify
a capacity roughly equal to the maximum number of elements the Map will contain (if known).

```lua
function map.create(c: integer): Map
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`c` – initial Map capacity.
    </ul>
  </dd>
  <dt>Returns
  <dd>The intialized `Map`
</dl>

Example:

Create an empty Map with space for 20,000 entries:

```lua
> local m = map.create(20000)
Map()
```


-------------------------------------------------------------------------------
#### map.size()

The number of (key, value) pair entries in the Map.

```lua
function map.size(m: Map): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`m` – The `Map` to get the size of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The number of entries in the map.
</dl>

Example:

```lua
> local m = map {a=1, b=2, c=3}
Map( "a" => 1, "b" => 2, "c" => 3 )

> map.size(m)
3
```


-------------------------------------------------------------------------------
#### map.pairs()

Get an iterator for all (key, value) pairs in a Map.

```lua
function map.pairs(m: Map): iterator
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`m` – The `Map` to iterate over.
    </ul>
  </dd>
  <dt>Returns
  <dd>The iterator function
</dl>

Example:

```lua
> local m = Map { a = 1, b = 2, c = 3 }
Map( "a" => 1, "b" => 2, "c" => 3 )
   
> for key, value in map.pairs(m) do
>   info("%s = %d", key, value)
> end
a = 1
b = 2
c = 3
```


-------------------------------------------------------------------------------
#### map.keys()

Get an iterator for all keys in a Map.

```lua
function map.keys(m: Map): iterator
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`m` – The `Map` to iterate over.
    </ul>
  </dd>
  <dt>Returns
  <dd>The iterator function
</dl>

Example:

```lua
> local m = Map { a = 1, b = 2, c = 3 }
Map( "a" => 1, "b" => 2, "c" => 3 )
   
> for key in map.keys(m) do
>   info("%s", key)
> end
a
b
c
```


-------------------------------------------------------------------------------
#### map.values()

Get an iterator for all values in a Map.

```lua
function map.values(m: Map): iterator
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`m` – The `Map` to iterate over.
    </ul>
  </dd>
  <dt>Returns
  <dd>The iterator function
</dl>

Example:

```lua
> local m = Map { a = 1, b = 2, c = 3 }
Map( "a" => 1, "b" => 2, "c" => 3 )
   
> for value in map.values(m) do
>   info("%d", value)
> end
1
2
3
```


-------------------------------------------------------------------------------
#### map.remove()

Remove an entry from a Map.

```lua
function map.remove(m: Map, key: string): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`m` – The `Map` to remove an entry from.
      <li>`key` – The key of the Map entry to remove.
    </ul>
  </dd>
</dl>

Example:

```lua
> local m = Map { a = 1, b = 2, c = 3 }
Map( "a" => 1, "b" => 2, "c" => 3 )

> map.remove(m, "b")
> m
Map( "a" => 1, "c" => 3 )
```


-------------------------------------------------------------------------------
#### map.clone()

Create a new Map as a shallow copy of a Map. All the keys and values are not copied, but still reference the original keys and values.

```lua
function map.clone(m: Map): Map
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`m` – The `Map` to clone.
    </ul>
  </dd>
  <dt>Returns
  <dd>A new `Map` containing all entries in the map.
</dl>

Example:

```lua
> local m1 = Map { a = 1 }
Map( "a" => 1 )
   
> local m2 = map.clone(m1)
Map( "a" => 1 )
   
> m2['a'] = 2
> m2
Map( "a" => 2 )
  
> m1
Map ( "a" => 1 )
```


-------------------------------------------------------------------------------
#### map.merge()

Merge two Maps, creating a new Map. When keys collide, call the merge function `op` to merge the values.

```lua
function map.merge(m1: Map, m2: Map, op: function): Map
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`m1` – A map to be merged.
      <li>`m2` – Another map to be merged.
      <li>`op` – The function to use when merging pairs with the same key.
    </ul>
  </dd>
  <dt>Returns
  <dd>A new `Map` containing all merged entries of the two maps.
</dl>

Where merge function `op` would take two values and return a resolved value:

```lua
function(a: Val, b: Val): Val
```

Example:

```lua
> local m1 = map { a = 1, b = 2 }
Map( "a" => 1, "b" => 2 )
   
> local m2 = map { a = 3 }
Map( "a" => 3 )
  
> map.merge(m1, m2, function (v1, v2)
>   return v1 + v2
> end)
Map( "a" => 4, "b" => 2 )
```
