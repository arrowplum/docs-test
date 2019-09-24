---
title: Transforming Records
description: Use the Aerospike Ruby client ability to scan records in the database and transform each record using user-defined functions (UDFs). 
---

Use the Aerospike Ruby client ability to scan records in the database and transform each record using user-defined functions (UDFs).

In SQL, you perform transformation on all rows in a table as an UPDATE statement without predicates:

```cpp
UPDATE test.demo
SET a = 1, b = 2
WHERE c = 3
```

Aerospike provides a similar capability, but allows you to transform records by applying a Record UDF function on each record. A Record UDF is applied to individual records. It can be provided as arguments, and can read and write the bins of a record and perform calculation. Refer to the  `QueryExecute` example in the _examples_ package.

### Defining Record UDFs

The following is the `processRecord` Record UDF defined in *record_example.lua*, where:
- The record contains two integer bins: *name1* and *name2*.
 - For *name1* even integers, add the value to the existing *name1* bin.
 - For *name1* integers with a multiple of 5, delete the *name2* bin.
 - For *name1* integers with a multiple of 9, delete the record. 

```lua
function processRecord(r, name1, name2, addValue)
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

After defining the UDF function, it must register with the Aerospike cluster. See [Register UDF](/docs/client/ruby/usage/udf/register.html).

### Initializing the Query Statement

To initialize a query statement: 

```ruby
stmt = Statement.new(namespace, set, [binName1])
stmt.filters << Filter.Range(binName1, begin, end)
```

{{#note}} If there is no value for `Filter`, the UDF function is applied to all records in the database. You must supply the corresponding secondary index for the specified bin. See [Manage Indexes](/docs/client/ruby/usage/query/sindex.html). {{/note}}

### Executing a UDF on a Record

To execute UDFs using `#execute_udf_on_query`:

```ruby
task = client.execute_udf_on_query(stmt, 
    'record_example', 'processRecord', [binName1, binName2, 100])
```

Where, 
- `stmt` &mdash; Specifies the set of records for the UDF.
- `record_example` &mdash; Specifies the UDF module.
- `processRecord` &mdash; Specifies the function to use on each record.
- These parameters passthrough to the UDF function: `name1`, `name2`, and `addValue`.

### Determining UDF Processing Status

To inform the application of job status, call `#wait_till_completed` in the task periodically:

```ruby
# to block until the UDF is run on all records in query
task.wait_till_completed
# task is completed successfully
```

