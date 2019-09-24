---
title: Deleting Records
description: Use the the Aerospike Node.js client to delete records in the Aerospike database. 
---

Use the Aerospike Node.js client `remove()` API to delete records in the Aerospike database.

This example defines a key object, and then deletes the corresponding record in the database:

```js
// Key of the record to be deleted
let key = new Aerospike.Key('test', 'demo', uid)

// Delete the record using the key k1
client.remove(key, function (error) {
  if (error) {
    console.log("error %s", error.message)
  }
})
```
