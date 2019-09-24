
 Lua: stream_ops Module

    Overview
    Usage
    Methods
        filter()
        map()
        aggregate()
        reduce()

Overview

Stream Operations are operations which may use to filter, transform or reduce the data in a stream. 
Usage

Stream Operations can only be applied to a stream. A stream is provided as the first argument of a Stream UDF, and you can add operations to the stream using using Lua's method syntax:
stream : op() : op()

The operations are not immediately applied to the stream. The operations are performed lazily, meaning only when necessary and when data is moving through the stream.
Methods
filter()

Returns a stream containing all elements satisfies the predicate p. 
filter: (p: A => Boolean) => Stream

The predicate p is a function that will test a value and returns true if the value should be included in the stream, otherwise it is excluded:
p: (a: A) => Boolean
Example:

Given a stream s, you can add the filter() operation as follows:
s : filter(p)

Where p would be a function like:
local function p(value)
  return value => 10 and value <= 100
end

Which assumes the value is an integer, and it should be in the inclusive range of 10 and 100.
map()

Returns a stream resulting from applying the function f to each element in the stream.
map: (f: A => B) => Stream

The function f accepts a value from the stream, and returns a new value:
f(a: A) => B
Example:

Given a stream s, you can add a map() operation:
s : map(f)

Where, f is a mapping function:
function f(rec)
  return rec['user_id']
end

Which selects the 'user_id' bin of a record, assuming it is reading a stream of records.
aggregate()

Aggregates the elements of this stream using the associative binary operator op.
aggregate: (x: B, op: (B,A) => B) => Stream

The argument x is a neutral element for the fold operation, which may be added to the result an arbitrary number of times.

The associative binary operator op will be called for each element in the stream. It will accept two argument, the first being either the default value x or the result of a prior call to op and current element. The return value may be used in subsequent calls to op. 
op: (b: B, a: A) => B
Example:

Given a stream s, we can add an aggregate operation:
s : aggregate(0, add)

Where the aggregate operation is given a neutral element of 0 (zero) and a aggregation function add, defined as:
local function add(a, b)
  return a + b
end

In which add() produces the sum of two values a and b. The value for a may be the neutral value 0 (zero) an arbitrary number of times.
reduce()

Reduces the elements of this stream using the associative binary operator op.
reduce: (op: (A, A) => A) => Stream[A]

The associative binary operator op should produce a value based on the two arguments. 
op: (a1: A, a2: A) => A
Example:

Given a stream s, we can add a reduce() operation:
s : reduce(r)

Where r is the reduction function, called for each element in the stream:
local function r(a,b)
  return a + b
end

Which assumes the reduce() is being applied to a stream of integers, and will reduce it to a single integer.



