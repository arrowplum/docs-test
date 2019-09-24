---
title: Multiple Operations
description: Use the Aerospike Node.js client to perform multiple operations in the Aerospike database.
---

Use the Aerospike Node.js client to perform multiple operations in the Aerospike database.

Aerospike provides the ability to perform multiple operations on a record within a single transaction. This feature allows modification of record bins, and then reads them back to the client within a single transaction. The application gets record status at transaction completion.

### Operations Available

Use the following operations on a record:

Operation | Description | Conditions
--- | --- | ---
write | Writes a value to a bin.   | 
read  | Reads the value of a bin.  | 
increment | Increments the integer value of a bin. |  Only integer values. 
append  | Appends a value to the value of a bin. |  Only bytes or strings. The value must be the same data type as the value in the bin. 
prepend | Prepends a value to the value of a bin. |  Only bytes or strings. The value must be the same data type as the value in the bin.
touch | Updates the generation value of the record.  | 

{{#note}}
Operations execute in the order specified by the client application.
{{/note}}

### Examples

These examples illustrate common operations.

#### Performing a Sequence of Operations

Increment-and-read is a common operation sequence that allows an application to use a counter and read the values after each increment.

This example demonstrates an increment-and-read operation:

```js
const Aerospike = require('aerospike')
const op = Aerospike.operations

// Array of operations to be performed on the record
let ops = [
  op.incr('views', 1),
  op.read('views')
]

// Key of the record on which the operation has to be performed
let key = new Aerospike.Key('app', 'pages', uid)

// operate on the record
Aerospike.connect(function (error, client) {
  if (error) {
    console.error('Error connecting to database: %s', error.message)
    process.exit(-1)
  }

  client.operate(key, ops, function (error, record) {
    if (error) {
      console.error('Error operating on record: %s', error.message)
    } else {
      let bins = record.bins
      console.info(bins.views) // post-increment value of 'views'
    }
  })
})
```

#### Touching a Record

Each record contains metadata that include its generation and time-to-live
(TTL) values. The generation is the version number of the record, which
increments with each update. The TTL is the time until the record expires.
These values are not modified on reads. If a record has a TTL of 5 minutes,
even if constantly read, it is not available after 5 minutes. To keep a record
from expiring, use the `touch` operation. If the `ttl` parameter is not specified,
the server will reset the record's TTL value to the default TTL value for the namespace.

This example reads three bins and touches a record to prevent it from expiring:

```js
// Get the operators object.
let op = Aerospike.operations

// Array of operations 
let ops = [
  op.touch(),
  # specify the ttl param to reset the record's TTL to a specific value, e.g.
  # op.touch(300),
  op.read('x'),
  op.read('y'),
  op.read('z')
]

// Key of the record
let key = new Aerospike.Key('app', 'pages', uid)

// Touch the record and read the bins x, y and z.
client.operate(key, ops, function (error, record) {
  if (error) {
    console.log(err.message);
  } else {
    let bins = record.bins
    let x = bins.x
    let y = bins.y
    let z = bins.z
  }
})
```
