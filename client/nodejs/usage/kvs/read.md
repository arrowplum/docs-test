---
title: Reading Records
description: Use the Aerospike Node.js client to read records in the Aerospike database. 
---

Use the Aerospike Node.js client API to read entire records, read selected bins in a record, and retrieve multiple records in one operation (batch read).

### Read Functions

Use any of the following functions to perform reads:

- `get()` &mdash; Read all record bins.
- `select()` &mdash; Read only specified record bins.
- `batchRead()` &mdash; Read a set of records from the database.
- `exists()` &mdash; Check record existence.

### Reading a Record

This example performs a full record read and on success, sends the results to the console:

```js
const Aerospike = require('aerospike')

// Specify the key of the record to be read.
let key = new Aerospike.Key('test', 'demo', 'key1')

// Retrieve the record using the key.
Aerospike.connect(function (error, client) {
  if (error) throw error
  client.get(key, function (error, record) {
    if (error) {
      switch (error.code) {
        case Aerospike.status.AEROSPIKE_ERR_RECORD_NOT_FOUND:
          console.log('NOT_FOUND -', key)
          break
        default:
          console.log('ERR - ', error, key)
      }
    } else {
      console.log('OK - ', record)
    }
    client.close()
  })
})
```

### Selecting Specific Bins of a Record

To specify the bins to read, build an array of strings where each string is a bin name, and then call `select()`. For explanations on how Aerospike uses bins, see [Data Model](/docs/architecture/data-model.html).

This example reads three bins _X_, _Y_, and _Z_ from the database:

```js
// Specify the key of the record to be read.
let key = new Aerospike.Key('test', 'demo', 'key1')

// Specify the bins of the record to be read.
let bins = [ 'X', 'Y', 'Z']

// Retrieve only the bins specified in bins variable
client.select(key, bins, function (error, record) {
  if (error) {
    console.error('Error: %s', error.message)
  }
})
```

### Reading a Batch of Records

To read a batch of records in a single read operation, the records must belong to the same namespace, and you must specify an array of key objects:

```js
// Specify the batch of keys of the records to be read.
let readKeys = [
  {key: new Aerospike.Key('test', 'demo', 'key1'), read_all_bins: true},
  {key: new Aerospike.Key('test', 'demo', 'key2'), read_all_bins: true},
  {key: new Aerospike.Key('test', 'demo', 'key3'), read_all_bins: true}
]

// Read the batch of records.
client.batchRead(readKeys, function (error, results) {
  if (error) {
    console.log('ERROR - %s', error.message)
  } else {
    results.forEach(function (result) {
      switch (result.status) {
        case status.AEROSPIKE_OK:
          let record = result.record
          console.log('OK - ', record)
          break
        case status.AEROSPIKE_ERR_RECORD_NOT_FOUND:
          console.log("NOT_FOUND - ", result.record.key)
          break
        default:
          console.log("ERROR - %d - %s", result.status, result.record.key)
      }
    }
  }
  client.close()
})
```

### Checking Existance of a Record

To check whether a record exists, use the `exists()` command:

```js
let key = new Aerospike.Key('test', 'demo', 'key1')
client.exists(key, function (error, result) {
  if (error) {
    // handle error
  } else if (result) {
    // record exists
  } else {
    // record does not exist
  }
})
```
