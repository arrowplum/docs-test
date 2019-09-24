---
title: Transforming Records
description: Use the Aerospike Go client to scan and transform records in the database using user-defined functions (UDFs). 
---

Use the Aerospike Go client to scan and transform records in the database using user-defined functions (UDFs).

In SQL, a transformation on all rows in a table is performed as an `UPDATE` statement with or without predicates:

```cpp
UPDATE test.demo
SET a = 1, b = 2
WHERE c = 3
```

The Aerospike Go client provides a similar capability to this SQL, but allows you to transform records by applying a function on each record (Record UDF). Record UDFs support arguments, and can read and write the bins of a record and perform calculation. See `QueryExecute` in the _examples_ package.

### Defining Record UDFs

This is the _processRecord_ Record UDF, defined in _record\_example.lua_: 

```lua
-- Record contains two integer bins, name1 and name2.
-- For name1 even integers, add value to existing name1 bin.
-- For name1 integers with a multiple of 5, delete name2 bin.
-- For name1 integers with a multiple of 9, delete record. 
function processRecord(r,name1,name2,addValue)
    local v = r[name1]

    if (v % 9 == 0) then
        aerospike:remove(r)
        return
    end

    if (v % 5 == 0) then
        r[name2] = nil
        aerospike:update(r)
        return
    end

    if (v % 2 == 0) then
        r[name1] = v + addValue
        aerospike:update(r)
    end
end
```

### Registering UDFs

Defined UDFs must register with the Aerospike cluster. See [Register UDF](/docs/client/go/udf/register.html).

### Initializing the Query Statement

To initialize a query statement:

```go
stmt := aerospike.NewStatement(namespace, set, binName1)
stmt.AddFilter(aerospike.NewRangeFilter(binName1, begin, end))
```

{{#note}} 
Without a specified filter, the UDF is applied to all records in the database.

The filter must have a corresponding secondary index on the specified bin. 

See [Manage Indexes](/docs/client/go/usage/query/sindex.html).
{{/note}}

### Executing UDFs on Records

To execute the UDF using `ExecuteUDF()`:

```go
task, err := client.ExecuteUDF(policy, stmt, 
    "record_example", "processRecord", aerospike.NewValue(binName1), 
    aerospike.NewValue(binName2), aerospike.NewValue(100))
```

Where,
 
- `stmt` &mdash; The set of records the UDF should be applied on.
- `record_example` &mdash; The UDF module.
- `processRecord` &mdash; The function to use on each record.

The next parameters passthrough to the UDF as parameters `name1`, `namee2`, and `addValue`.

### Waiting for the UDF

To inform the application of job completion, read `OnComplete()`:

```go
// to block until the UDF is run on all records in query
for err := range task.OnComplete(); err != nil {
    // deal with the error here
}
// task is completed successfully
```

