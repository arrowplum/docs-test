---
title: Stream UDF 
description: Use Aerospike User-Defined Functions (UDFs) to extend database functionality and performance.
assets: /docs/guide/assets
---

Use Aerospike Stream UDFs to filter, transform, and aggregate query results in a distributed fashion. Stream UDFs can process a stream of data by defining a sequence of operations to perform. Stream UDF aggregations run in a distributed fashion to exploit the power of multiple machines.

Stream UDFs perform read-only operations on a collection of records. The Stream UDF system allows a MapReduce style of programming, which is used for common, highly parallel jobs such as *word count*&mdash;each row is accessed and a list of words and their counts are emitted. The top results are calculated in a reduce phase. For simple aggregations where counters are simply incremented in a context instead of continually creating and destroying objects, Aerospike provides optimal implementation.

{{#note}}
See [Developing Stream UDFs](/docs/udf/developing_stream_udfs.html).
{{/note}}

## Operators

Aerospike Stream UDFs support these operators:

- `filter`&mdash;Filter data in the stream that satisfies a predicate.
- `map`&mdash;Transform a piece of data.
- `aggregate`&mdash;Reduce partitions of data to a single value.
- `reduce`&mdash;Allow parallel processing of each group of output data.

These operators are considered primitives and can build complex operations (see [Developing Stream UDFs](/docs/udf/developing_stream_udfs.html)).

### filter

Filter values in the stream using the predicate `p`. `p` tests each value in the stream and returns *true* if the value should be passed through or *false* to drop the value.

**Example**

```coffeescript
filter(p: (a: Value) -> Boolean) -> Stream
```

**Parameters**

- `p`&mdash;The predicate to apply to each value in the stream.

**Returns**

A stream of filtered values.

### map

Transforms a value to a different value using the identify function `f`.

```coffeescript
map(f: (a: Value) -> Value) -> Stream
```

**Parameters**

- `f`&mdash;The identify function to apply to each value in the stream.

**Returns**

A stream of the transformed values.

### reduce

Reduce the values in the stream to a single value and apply the associative binary operator `op` to each value. The `op` return value is the parameter `a` for subsequent calls to `op`.

```coffeescript
reduce(op: (a: Value, b: Value) -> Value) -> Stream
```

**Parameters**

- `op`&mdash;The associative binary operation to apply to each value in the stream. 

**Returns**

A stream containing a single value.


### aggregate

Takes the current value (parameter `x`) and the next value in the input stream, and returns the aggregate value (parameter `a`).

```coffeescript
aggregate(x: Value, op: (a: Value, b: Value) -> Value) -> Stream
```

**Parameters**

- `x`&mdash;The initial (neutral) value passed to the operator `op`. 
- `op`&mdash;The identify function to apply to each value in the stream.

**Returns**

A stream containing a single value.

## Examples

{{#note}}
See [Stream UDF Examples](/docs/udf/examples/stream_udf_examples.html).
{{/note}}


## References

- [UDF Developer Guide](/docs/udf/udf_guide.html)
- [UDF Library API Reference Guide](/docs/udf/api_reference.html).
