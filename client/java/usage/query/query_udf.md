---
title: Transforming Records
description:  Aerospike provides the ability to scan records and transform each record using UDFs. 
---

Aerospike provides the ability to query records and transform each record using UDFs. 

In SQL, you perform transformation on all rows in a table as an UPDATE statement without predicates:

```cpp
UPDATE test.demo
SET a = 1, b = 2
WHERE c = 3
```

Aerospike provides a similar capability, but allows you to transform records by applying a Record UDF function on each record. A Record UDF is applied to individual records. It can be provided as arguments, and can read and write the bins of a record and perform calculation. Refer to the  `QueryExecute` example in the _examples_ package.

### Defining the Record UDF

The following is the `processRecord` Record UDF defined in `record_example.lua`: 

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

After defining the UDF function, it must register with the Aerospike cluster. See [Register UDF](/docs/client/java/usage/udf/register.html).

### Initializing the Query Statement

To initialize a query statement: 

```java
Statement stmt = new Statement();
stmt.setNamespace(params.namespace);
stmt.setSetName(params.set);
stmt.setFilters(Filter.range(binName1, begin, end));
```

{{#note}} If there is no value for `Filter`, the UDF function is applied to all records in the database. You must supply the corresponding secondary index for the specified bin. See [Manage Indexes](/docs/client/java/usage/query/sindex.html). {{/note}}

### Executing a UDF on a Record

To execute UDFs, call `execute()`:

```java
ExecuteTask task = client.execute(params.policy, stmt, 
    "record_example", "processRecord", Value.get(binName1), Value.get(binName2), Value.get(100));
```

Where, 
- `stmt` &mdash; Specifies the set of records for the UDF.
- `record_example` &mdash; Specifies the UDF module.
- `processRecord` &mdash; Specifies the function to use on each record.
- These parameters passthrough to the UDF function: `name1`, `name2`, and `addValue`.

### Determining UDF Processing Status

To inform the application of job status, call `waitTillComplete()` in the task periodically.
