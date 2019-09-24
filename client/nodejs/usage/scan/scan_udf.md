---
title: Transforming Records
description: Use the Aerospike Node.js client to scan the records in the database and transform each record using a user-defined function (UDF). 
---

Use the Aerospike Node.js client to query the records in the database and transform each record using a user-defined function (UDF).

In SQL, you perform a transformation on all rows in a table as an `UPDATE` statement without predicates:

```sql
UPDATE test.demo SET a = 1, b = 2 WHERE c = 3
```

Aerospike allows you to transform records by applying a function on each record. This **Record UDF** is applied to individual records. It accepts arguments and can read and write record bins and perform calculation. 

### Defining the Record UDF

The following is the `processRecord` Record UDF that is defined in *record_example.lua*: 

```lua
-- Record contains two integer bins, name1 and name2.
-- For name1 even integers, add value to existing name1 bin.
-- For name1 integers with a multiple of 5, delete name2 bin.
-- For name1 integers with a multiple of 9, delete record. 
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

### Registering the UDF

UDF modules must register with the Aerospike  (see [Register UDF](/docs/client/nodejs/usage/udf/manage.html)).

# Initializing the Scan

To initialize a scan:

```js
var scan = client.scan('test', 'demo')
```

### Executing a UDF on Records

To execute the UDF use the `Scan#background` method. The scan will be
executed on the server in the background, i.e. the records will not be returned
to the client. The callback function returns a `Job` instance which can be used
to check on the status of the job.

```js
scan.background('record_example', 'processRecord', ['binName1', 'binName2', 100], function (error, job) {
  if (error) throw error
  job.waitUntilDone(function (error) {
    // background job has completed
  })
})
```
