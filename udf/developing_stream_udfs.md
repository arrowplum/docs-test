---
title: Developing Stream UDFs
description: Learn how start writing an application using the aggregation functionality using Stream User-Defined Functions in Aerospike in order to perform a sequence of operations on a set of records. 
assets: /docs/udf/assets
---

### Overview

Stream UDFs (also called Aggregations) in Aerospike provide the ability
to perform a sequence of operations on a set of records.
Currently, the stream UDFs are allowed only on a secondary index query
result set.
For a high-level idea about this feature please refer to the
[Aggregations Feature Guide](/docs/guide/aggregation.html).
This page will help the developer to get started to write an application
using the aggregation functionality.
For more examples please refer to the
[Stream UDF Examples](/docs/udf/examples/stream_udf_examples.html).

To start writing Stream UDFs, one should be aware of the following concepts.
Please refer to the corresponding documents.

* Creating secondary index and running queries on them (Ref: [Query Guide](/docs/guide/query.html)).
* Creating UDF and Registering the UDF
(Ref: [UDF Developer Guide](/docs/udf/udf_guide.html),
 Ref: [Managing UDFs](/docs/udf/managing_udfs.html)).
* API reference for LUA stream-ops module (Ref: [API Stream](/docs/udf/api/stream.html)).

Currently, Stream UDF supports the following types of operations.
The user has to write one or more of these functions depending on the
application's requirements.

* filter — retain data in the stream that satisfies a predicate.
* map — transform a piece of data into.
* aggregate — aggregates a stream of data into a single aggregate value.
* reduce —  reduce produces a value by combining two like values.

### Basics

The main difference between a Record UDF and a Stream UDF is that the
Record UDF returns a value,
whereas the Stream UDF returns a stream definition.
The stream definition contains a series of functions that should be applied on the data
that is fed to the stream.
**The stream-UDF is read-only**.
If an attempt is made to update a record by any function from [Aerospike lua module](/docs/udf/api/aerospike.html) in the stream definition,
the function will be a no-op and return failure.
It is like the pipe mechanism in linux where the output of one command can be fed
into the next command in a chain:

	cat peoplelist.txt | grep "USA" | grep "California" | wc -l

However, in case of stream UDF,  the user has to indicate upfront the type of function because each type of operation is treated differently by the stream sub-system. Each function will have an input stream and an output stream. Data gets fed to the function from the input stream and the result will be put on its output stream. The output stream of a function becomes an input stream to the next function in the chain. A typical stream UDF looks as below:

```lua
function my_stream_udf(s)
    local m = map()
    return s : filter(my_filter_fn) : map(my_map_fn): aggregate(m, my_aggregate_fn): reduce(my_reduce_fn)
end
```

It is legal to specify multiple functions of any type in any order as long as they can work together.
For example, the example below is a legal stream.

```lua
function my_complex_stream_udf(s)
    return s : filter(my_filter1) : map(my_map1) : filter(my_filter2) : map(my_map2) : map(my_map3) : reduce(my_reduce)
end
```

The stream UDFs execute in two phases.

* Cluster-side : First the stream UDF is executed on all the nodes of the cluster. The result from each node (after applying first reduce) is sent to the application which triggered the UDF.
* Client-side: When the nodes send their result to the application, the client layer will do the final aggregation. Execution at client-side will start from first reduce function and send the result to application.

![]({{book.assets}}/stream.png)

The UDF file should be present both on the cluster nodes as well as the client nodes as the final phase of reduction happens on the client side.

### The Filter Operation

The **filter operation** will filter values from the stream.
The filter operation accepts a single argument, the filter function, e.g. return s : filter(my_filter1).

The **filter function** accepts the current value from the stream and should return true or false, where true indicates the value should continue down the stream.

A typical filter function looks as below.
A query will feed records in to the stream.
The filter operation will apply the filter function for each record in the stream.
In the example, the filter function allows records containing "males" older than "18" years to be passed down the stream.

```lua
local function my_filter_fn(rec)
    if rec['age'] > 18 and rec['sex'] == 'male' then
        return true
    else
        return false
    end
end
```

### The Map Operation

The **map operation** will transform values in the stream.
The map operation accepts a single argument, the map function, e.g. return s : map(my_map1).

The **map function** accepts the current value from the stream, and should return
a new value to replace the current value in the stream. The type of the return value must be one of those supported by the database: integer, string, list, and map.

A typical map function looks as below:

```lua
local function my_map_fn(rec)
    local result = map()
    result['name'] = rec['name']
    result['city'] = rec['city']
    return result
end
```

### The Aggregate Operation

Aggregate operation aggregates a stream of data into a single aggregate value.
The **aggregate operation** accepts two arguments and returns one value.
The first argument is the aggregated value where the results of the aggregation are stored.
The second argument is an aggregation function which will aggregate the values coming from the input stream and store in the aggregated value (the first argument).

The **aggregation function** takes two arguments, the aggregate value and the next value from the input stream.
It should return a single value whose type must be one of those supported by the database: integer, string, list, and map.
The aggregate value and the return value should be of same type.
Each call to the aggregate function should populate the aggregate value with data contained in the next value from the input stream.

A typical aggregate function looks as below.
In the example, it is trying to do a group-by kind of operation where it is trying to find the number of citizens in each city. You will notice reduce is called after aggregate, which will perform a final reduction on the data

```lua
local function my_aggregation_fn(aggregate, nextitem)
    -- If the count for the city does not exist, initialize it with 1
    -- Else increment the existing counter for that city
    aggregate[nextitem['city']] = (aggregate[nextitem['city']] or 0) + 1
    return aggregate
end
```

On close observation one can realize that the aggregate function is different from filter and map type of functions.
Both filter and map takes one element as from its input stream and returns one element to its output stream.
Whereas, the aggregate function will take a bunch of elements from its input stream (across multiple invocations)
but will return only one element to its output stream.
So, putting two aggregate functions in a row is not of much benefit because the output of the first aggregate function will only emit single element which can be consumed by the next aggregate function.

It should also be noted that the accumulated aggregate value can grow quite large.
It is possible, for example, to simulate the SQL Select DISTINCT function by accumulating values
in a map and then dumping the map at the end.

	NOTE:: This example will be added to examples soon.

### The Reduce Operation

The **reduce operation** will reduce values in the stream to a single value.
The reduce operation accepts a single argument, the reduce function.

The **reduce function** accepts two values from the stream and returns a single value to be fed back into the function as the first argument. The reduce function should be commutative. The two arguments of the reduce function and the return value should be of the same type.  The type of the return value must be one of those supported by the database: integer, string, list, and map.

One main characteristic of reduce function is that it executes both on the server nodes as well as the client side (in application instance). Each node first runs the data stream through the functions defined in the stream definition. The end result of this is sent to the application instance. The application gets results from all the nodes in the cluster. The client layer in it does the final reduce using the reduce function specified in the stream. So, the reduce function should be able to aggregate the intermediate aggregated values (coming form the cluster nodes). If there is no reduce function, the client layer simply passes all the data coming from the nodes to the application.

```lua
local function my_merge_fn(val1, val2)
    return val1 + val2
end
 
local function my_reduce_fn(global_agg_value, next_agg_value)
    -- map.merge is a library function will call the my_merge_fn for each key and returns a new map.
    -- my_merge_fn will be passed the values of the key in two maps as arguments.
    -- It stores the result of merge function against the key in the result map
    return map.merge(global_agg_value, next_agg_value, my_merge_fn)
end
```

#### Data Types

In case of stream UDFs, there will be a series of functions and the output
of one function goes as input of the next function.
So, there should be an agreement between a function and the next function
in the chain.
The data type returned by a function should be of a type that can be
consumed by the next function in the chain.
The developer has to take care of this.
If this agreement is broken, the stream UDF execution may fail due to
exception.

The data that is fed to the first function in the stream will be of
type 'record'.
It should be capable of consuming it.
To see how to consume a 'record' refer to the
[UDF guide](/docs/udf/udf_guide.html).

More on the data types of functions in section below.

### Extra Arguments Using Closures

The examples above uses hard-coded values for filtering and transformation.
It will not be practical to write a whole lot of filter/map/reduce function
for each specific use case, for example, one filter function for
`age > 18` and one more for `age > 25`.
One will want to write a generic function and use it in different ways by
passing arguments to it.
To pass arguments to these filter/map/reduce type of functions we should use
the concept of a closure.
In short, closure is a function which will capture the environment in which
it is created.

An example will help illustrate:

```lua
local function my_age_filter(minimum_age)
    return function(rec)
        if rec['age'] > minimum_age
            return true
        else
            return false
    end
end
 
function my_stream_udf(stream, minimum_age)
    local myfilter = my_age_filter(minimum_age)
    return stream : filter(myfilter) : aggregate(0, myreducer)
end
```

The arguments can be passed when invoking the stream UDF.
Refer to the [UDF guide](/docs/udf/udf_guide.html) how to pass arguments to
it during its invocation.

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

Each subsequent argument is defined by the UDF and must be one of the types supported by the database.

### Return Values

A Stream UDF should always return the stream argument augmented with operations.

