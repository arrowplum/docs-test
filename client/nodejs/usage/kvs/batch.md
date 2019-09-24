---
title: Batch Read Records
description: Run batch record reads using the Aerospike Node.js client on the Aerospike database.
categories:
  - aerospike-client-nodejs
tags:
  - aerospike-client-nodejs
  - nodejs
---

In addition to reading single record each transaction, you can perform multiple
record reads from the cluster in one transaction using the `client.batchRead()`
command 

To continue, the client must have an [established the cluster connection](/docs/client/nodejs/usage/connect).

### Batch Requests

To read a batch of records:

```js
function run (client) {
  var readKeys = keys.map(function (key) { return {key: key, read_all_bins: true} })
  client.batchRead(readKeys, function (error, results) {
    assertOk(error, 'Batch read keys')
    console.log(results)
    iteration.next(run, client)
  })
}

function assertOk (error, message) {
  if (error) {
    console.error('ERROR - %s: %s [%d]\n%s', message, error.message, error.code, error.stack)
    throw error
  }
}

Aerospike.connect(config, function (error, client) {
  assertOk(error, 'Connect to database')
  run(client)
})
```

### Checking that Records Exists

To check that records exist in the Aerospike database:

```js
function run (client) {
  var readKeys = keys.map(function (key) { return {key: key, read_all_bins: false} })
  client.batchRead(readKeys, function (error, results) {
    assertOk(error, 'Batch read keys')
    console.log(results)
    iteration.next(run, client)
  })
}

Aerospike.connect(config, function (error, client) {
  assertOk(error, 'Connect to database')
  run(client)
})
```

### Initializing Keys to Read

To initialize the keys to read:

```js
var argv = argp.argv
var keys = argv._.map(function (key) {
  return new Key(argv.namespace, argv.set, key)
})
```

{{#note}}
The keys must all belong to the same namespace.
{{/note}}
