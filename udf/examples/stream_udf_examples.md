---
title: Stream UDF Examples
description: Stream UDF Examples
---

Several Stream UDF examples have been created to
demonstrate the various characteristics of Stream UDFs.
We start with some general comments on Stream UDFs and developers may find useful.

### General Comments on Stream UDFs

* Although the presentation of Stream UDFs often includes the standard `filter() | map() | reduce()` arrangement, in fact any combination of those functions can be used.

* A Stream UDF is function which is applied to a stream of data.
 The first argument will always be a Lua: stream_ops Module.
The return value must be the Lua: stream_ops Module, optionally augmented with operations.

The following is a simple Stream UDF in Lua:

```lua
function my_stream_udf(stream)
  return stream
end
```

### Arguments

A Stream UDF should have one or more arguments.

The first arguments will always be a Lua: stream\_ops Module, which defines the operations to be invoked on the data in a stream.

Each subsequent argument is defined by the UDF and must be one of the types supported by the database: integer, string, list and map.
Return Values

A Stream UDF should always return the stream argument augmented with operations.

### The Stream UDF Examples

* [Stream UDF: Simple Statistics Example](/docs/udf/examples/stream_udf_stats.html)
* [Stream UDF: Word Count](/docs/udf/examples/stream_udf_word_count.html)
* [Stream UDF: String Starts With](/docs/udf/examples/stream_udf_starts_with.html)
