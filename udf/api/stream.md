---
title: stream
description: Learn how to use Aerospike's Stream API, a sequence of values, to filter, transform or reduce the values within the Aerospike database.
---

Stream is a sequence of values, which may have operations to filter, transform or reduce the values.

### Usage

A stream is provided as the first argument of a Stream UDF, and you can add operations to the stream using using Lua's method syntax:

```lua
function a_stream_udf(stream)
  return stream : op() : op()
end
```

The above code example, is a Stream UDF, which is given a stream as the first argument, and returns the stream with additional operations.

The operations are not immediately applied to the stream. The operations are performed lazily, meaning only when necessary as data is pushed into the stream.

### Methods

-------------------------------------------------------------------------------
#### stream:filter()

Returns a stream containing all elements satisfies the predicate function `p`. 

```lua
function stream:filter<A>(p: function): Stream<A>
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`p` – The predicate function to be applied to each value in the stream.
    </ul>
  </dd>
  <dt>Returns
  <dd>A stream of filtered values.
</dl>

The predicate function `p` will test a value and returns true if the value should be included in the stream, otherwise it is excluded. The predicate function `p` can be described as:

```lua
function<A>(a: A): Boolean
```

Example:

Given a stream `s`, you can add the `filter()` operation as follows:

```lua
s : filter(p)
```

Where `p` would be a predicate function like:

```lua
local function p(value)
  return value => 10 and value <= 100
end
```

Which assumes the value is an integer, and it should be in the inclusive range of 10 and 100.


-------------------------------------------------------------------------------
#### stream:map()

Returns a stream resulting from applying the transform function `f` to each element in the stream.

```lua
function stream:map<A,B>(op: function): Stream<B>
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`op` – The tranform operator to be applied to each value in the stream.
    </ul>
  </dd>
  <dt>Returns
  <dd>A stream of transformed values.
</dl>

The transform operator `op` accepts a value from the stream, and returns a new value which must be one of the types supported by the database: integer, string, list, and map. The transform operator `op` can be described as:

```lua
function<A,B>(a: A): B
```

Example:

Given a stream `s`, you can add a `map()` operation:

```lua
s : map(f)
```

Where, `f` is a transform function, that extracts the `user_id` from the `rec` argument:

```lua
function f(rec)
  return rec['user_id']
end
```

Which selects the `user_id` bin of a record, assuming it is reading a stream of records.


-------------------------------------------------------------------------------
#### stream:aggregate()

Folds the elements of this stream using the associative binary operator op, into the value `x`.

```lua
function stream:aggregate<A,B>(x: B, op: function): Stream<B>
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`x` – A neutral element for the fold operation, which may be added to the result an arbitrary number of times.
      <li>`op` – The binary operator to aggregate values in the stream.
    </ul>
  </dd>
  <dt>Returns
  <dd>A stream of aggregated values.
</dl>

The argument `x` is a neutral element for the fold operation, which may be added to the result an arbitrary number of times.

The binary operator `op` will be called for each element in the stream. It will accept two argument, the first being either the default value x or the result of a prior call to `op` and current element. The return value may be used in subsequent calls to `op`, and must be one of the types supported by the database: integer, string, list, and map. The binary operator `op` can be described as:

```lua
function<B,A>(b: B, a: A): B
```

Example:

Given a stream `s`, we can add an `aggregate()` operation:

```lua
s : aggregate(0, add)
```

Where the aggregate operation is given a neutral element of 0 (zero) and a binary operator `add()`, defined as:

```lua
local function add(a, b)
  return a + b
end
```

In which `add()` produces the sum of two values a and b. The value for a may be the neutral value 0 (zero) an arbitrary number of times.


-------------------------------------------------------------------------------
#### stream:reduce()

Reduces the elements of this stream using the associative binary operator op.

```lua
function stream:reduce<A>(op: function): Stream<A>
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`op` – The binary operator to reduce values in the stream.
    </ul>
  </dd>
  <dt>Returns
  <dd>A stream of reduced values.
</dl>

The associative binary operator `op` should produce a value based on the two arguments. The associative binary operator `op` can be described as:

```lua
function<A>(a1: A, a2: A): A
```

Example:

Given a stream `s`, we can add a `reduce()` operation:

```lua
s : reduce(r)
```

Where `r` is the reduction function, called for each element in the stream:

```lua
local function r(a,b)
  return a + b
end
```

Which assumes the `reduce()` is being applied to a stream of integers, and will reduce it to a single integer.


