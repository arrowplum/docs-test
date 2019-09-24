---
title: list
description: Aerospike's list modules introduce a List type and functions to Lua for a consistent interface for working with a sequence of values, which is not provided by Lua Tables.
---

### Usage

The list modules introduces a List type and functions to Lua. It provides a consistent interface for working with a sequence of values, which is not provided by Lua Tables.

Aerospike Lua list objects are usually returned in Aerospike functions, and set as values in the Aerospike database.

To create an empty List, you can simply call the list() function (the initial capacity of the List will be 5):

```lua
> local l = list()
List()
```

To create an empty List with a specific initial capacity, you can call the list.create() function:

```lua
> local l = list.create(100)
List()
```

If you want to initialize a List with values, then you can call the list() function with an indexed table:

```lua
> local l = list {1,2,3,4}
List(1,2,3,4)
```

You can then access elements like a Lua Table:

```lua
> l[1]
1
> l[1] = 5
> l[1]
5
```

Note that because assigning a nil value in a Lua table deletes the entry from the table, it is not possible
to initialize a List containing nil values via the list() function.

A list must only contain values of the following type:

- integer
- string
- [bytes](/docs/udf/api/bytes.html)
- [list](/docs/udf/api/list.html)
- [map](/docs/udf/api/map.html)

Placing other Lua types - for example, functions or tables - will result in run-time errors when you attempt to save them in the database.

### Functions

-------------------------------------------------------------------------------
#### list()

Creates a new List.

```lua
function list(t: Table?): List
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`t` – (optional) Lua table containing the initial values.
    </ul>
  </dd>
  <dt>Returns
  <dd>The initialized `List`
</dl>

Examples:

To create an empty List.

```lua
> local l = list()
List()
```

To create a new List and initialize it via a Lua Table

```lua
> local l = list {1,2,3}
List(1,2,3)
```


-------------------------------------------------------------------------------
#### list.create()

Creates a new List with a given initial capacity and (optional) capacity step size.

```lua
function list.create(c: integer, s: integer?): List
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`c` – initial capacity of the List.
      <li>`s` – (optional) whenever the List must grow to accomodate more values, its capacity
will increase by this amount.  The default value is 10.
    </ul>
  </dd>
  <dt>Returns
  <dd>The initialized `List`
</dl>

Examples:

To create an empty List with an initial capacity of 20

```lua
> local l = list.create(20)
List()
```

To create an empty List with an initial capacity of 50 and a step size of 25

```lua
> local l = list.create(50, 25)
List()
```


-------------------------------------------------------------------------------
#### list.size()

The number of elements in a List.

```lua
function list.size(l: List): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to get the size of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The number of entries in the list.
</dl>

Example:

```lua
> local l = list {1,2,3}
List(1,2,3)
   
> list.size(l)
3
```


-------------------------------------------------------------------------------
#### list.iterator()

Get an iterator for all elements in a List.

```lua
function list.iterator(l: List): iterator
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to get the iterator of.
    </ul>
  </dd>
  <dt>Returns
  <dd>An iterator function.
</dl>

Example:

```lua
> local l = list {1,2,3}
List(1,2,3)

> for value in list.iterator(l) do
>    info("%s", tostring(value))
> end
1
2
3
```


-------------------------------------------------------------------------------
#### list.insert()

Insert a value into the List at a specified position.

```lua
function list.insert(l: List, i: Integer, v: Val): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to insert the value into.
      <li>`i` – The list index at which to insert the value.
      <li>`v` – The value to insert into the list.
    </ul>
  </dd>
</dl>

Example:

```lua
> local l = list {1,2,3,4}
List(1,2,3,4)

> list.insert(l, 3, 55)
List(1,2,55,3,4)   
```


-------------------------------------------------------------------------------
#### list.append()

Append a value to the end of the List.

```lua
function list.append(l: List, v: Val): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to append the value to.
      <li>`v` – The value to append to the list.
    </ul>
  </dd>
</dl>

Example:

```lua
> local l = list {1,2,3}
List(1,2,3)

> list.append(l, 4)
List(1,2,3,4)   

> list.append(l, 5)
List(1,2,3,4,5)
```


-------------------------------------------------------------------------------
#### list.prepend()

Prepend a value to the beginning of the List.

```lua
function list.prepend(l: List, v: Val): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to prepend the value to.
      <li>`v` – The value to prepend to the list.
    </ul>
  </dd>
</dl>

Example:

```lua
> local l = list {1,2,3}
List(1,2,3)

> list.prepend(l, 4)
List(4,1,2,3)

> list.prepend(l, 5)
List(5,4,1,2,3)
```


-------------------------------------------------------------------------------
#### list.take()

Select the first n elements of the List

```lua
function list.take(l: List, n: Integer): List
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to extract values from.
      <li>`n` – The number of values to select from the beginning of the list.
    </ul>
  </dd>
  <dt>Returns
  <dd>A new list containing the first `n` values of the list `l`.
</dl>

Example:

```lua
> local l = list {1,2,3}
List(1,2,3)
  
> list.take(l, 2)
List(1,2)
```


-------------------------------------------------------------------------------
#### list.remove()

Remove an element from the List at a specified position.

```lua
function list.remove(l: List, i: Integer): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to remove the value from.
      <li>`i` – The index of the list value to remove.
    </ul>
  </dd>
</dl>

Example:

```lua
> local l = list {1,2,3}
List(1,2,3)
  
> list.remove(l, 2)
List(1,3)
```


-------------------------------------------------------------------------------
#### list.drop()

Select all elements except the first n elements of the List.

```lua
function list.drop(l: List, n: Integer): List
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to extract values from.
      <li>`n` – The number of values to skip from the beginning of the list.
    </ul>
  </dd>
  <dt>Returns
  <dd>A new list containing the values after the first `n` values of the list `l`.
</dl>

Example:

```lua
> local l = list {1,2,3}
List(1,2,3)
  
> list.drop(l, 2)
List(3)
```


-------------------------------------------------------------------------------
#### list.trim()

Remove all elements at and beyond a specified position in the List.

```lua
function list.trim(l: List, i: Integer): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to trim elements from.
      <li>`i` – The index from which to trim the list.
    </ul>
  </dd>
</dl>

Example:

```lua
> local l = list {1,2,3,4,5}
List(1,2,3,4,5)
  
> list.trim(l, 4)
List(1,2,3)
```


-------------------------------------------------------------------------------
#### list.clone()

Create a new List as a shallow copy of another List. All elements are references to the elements of the original List, not copies.

```lua
function list.clone(l: List): List
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l` – The `List` to clone
    </ul>
  </dd>
  <dt>Returns
  <dd>A new list containing all values of the list `l`.
</dl>


Example:

```lua
> local l1 = list { 1, 2, 3 }
List(1,2,3)
    
> local l2 = list.clone(l1)
List(1,2,3)
    
> list.append(l1, 4)
> l1
List(1,2,3,4)
    
> list.append(l2, 5)
> l2
List(1,2,3,5)
```


-------------------------------------------------------------------------------
#### list.concat()

Append all elements of one list, in order, to another list.

```lua
function list.concat(l1: List, l2: List): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l1`  – The `List` to append to.
      <li>`l2`  – The `List` to be appended.
    </ul>
  </dd>
</dl>


Example:

```lua
> local l1 = list { 1, 2, 3, 4 }
List(1,2,3,4)
    
> local l2 = list { 8, 9, 10 }
List(8,9,10)
    
> list.concat(l1, l2)
> l1 
List(1,2,3,4,8,9,10)
> l2 
List(8,9,10)
```


-------------------------------------------------------------------------------
#### list.merge()

Merge the elements of given two lists and return the combined list.

```lua
function list.merge(l1: List, l2: List): List
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`l1`  – One of the `List` to  be merged
      <li>`l2`  – Another `List` to  be merged
    </ul>
  </dd>
  <dt>Returns
  <dd>A new list containing all the values in the lists `l1` and `l2`.
</dl>


Example:

```lua
> local l1 = list { 1, 2, 3 }
List(1,2,3)
    
> local l2 = list { 5, 6, 7 }
List(5, 6, 7)
    
> l3 = list.merge(l1, l2)
> l3 
List(1,2,3,5,6,7)
    
```
