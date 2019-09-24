---
title: Writing Records
description: Use the Aerospike Node.js client to write records to the Aerospike database.
---

Use the Aerospike Node.js client `put()` API to write records to the Aerospike database.

### Defining the Key

To store a record, you must define a key as the record ID. 

This example creates the _uid_ key value to store in namespace _test_ within set _demo_:

```js
const Aerospike = require('aerospike')
let key = new Aerospike.Key('test', 'demo', uid)
```

### Specifying Record Data

Define a record using a JavaScript object, where the fields of the object represent each bin in the record.

This example writes five bins: _uid_, _name_, _dob_, _friends_, and _avatar_:

```js
// Record to be written to the database
let bins = {
    uid:     1000,                // integer data stored in bin called "uid"
    name:    'user_name',         // string data stored in bin called "user_name"
    dob:     { mm: 12, dd: 29, yy: 1995},  // map data stored (msgpack format) in bin called "dob" 
    friends: [1001, 1002, 1003],  // list data stored (msgpack format) in bin called "friends"
    avatar:  Buffer.from([0xa, 0xb, 0xc])   // blob data stored in a bin called "avatar"
}
```

### Storing the Record

To store the record in the database using `put()`:

```js
// Put the record to the database.
client.put(key, bins, function (error) {
  if (error) {
    console.log('error: %s', error.message)
  } else {
    console.log('Record written to database successfully.')
  } 
})
```
