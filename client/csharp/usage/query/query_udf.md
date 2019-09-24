---
title: Transforming Records
description: Use the Aerospike C# client to scan records in the database and transform each record using UDFs. 
---

Use the Aerospike C# client to scan records in the database and transform each record using UDFs.

In SQL, you perform transformation on all rows in a table as an `UPDATE` statement without predicates:

```cpp
UPDATE test.demo
SET a = 1, b = 2
WHERE c = 3
```

Aerospike provides a similar capability, but allows you to transform records by applying a Record UDF function on each record. A Record UDF is applied to individual records. It can be provided as arguments, and can read and write the bins of a record and perform calculation. 

### Defining the Record UDF

The following example is the `processRecord` Record UDF defined in `record_example.lua`: 

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

After defining the UDF function, it must register with the Aerospike cluster. See [Register UDF](/docs/client/csharp/usage/udf/register.html).

### Initializing a Query Statement

To initialize a query statement: 

```cs
Statement stmt = new Statement();
stmt.SetNamespace(params.namespace);
stmt.SetSetName(params.set);
stmt.SetFilters(Filter.Range(binName, begin, end));
```

{{#note}} If there is no value for `Filter`, the UDF function is applied to all records in the database. You must supply the corresponding secondary index on the specified bin. See [Manage Indexes](/docs/client/csharp/usage/query/sindex.html). {{/note}}

### Executing a UDF on a Record

To execute UDFs, call `Execute()`:

```cs
ExecuteTask task = client.Execute(params.policy, stmt, 
    "record_example", "processRecord", Value.Get(binName1), Value.Get(binName2), Value.Get(100));
```

Where,

- `stmt` &mdash; Specifies the set of records for the UDF.
- `record_example` &mdash; Specifies the UDF module.
- `processRecord` &mdash; Specifies the function to use on each record.
- These parameters passthrough to the UDF function: `binName1`; `binName2`; `addValue`.
 
### Determining UDF Processing Status

To inform the application of job status, call `Wait()` in the task periodically.
