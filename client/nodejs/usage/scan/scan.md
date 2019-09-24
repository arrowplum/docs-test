---
title: Scan Records
description: Use the Aerospike Nodejs client to scan all records in a specified namespace and set. 
---

Use the Aerospike Node.js client to scan all records in a specified namespace and set.

### Scanning Records

This example counts the number of records returned in the scan:

```js
var scan = client.scan('test', 'demo')
scan.concurrent = true
scan.nobins = false

var recordCount = 0
var stream = scan.foreach()
stream.on('data', function (record) {
  recordCount++
  if (recordCount % 1000 === 0) {
    console.log('%d records scanned', recordCount)
  }
})
stream.on('error', function (error) {
  console.error('Error while scanning: %s [%d]', error.message, error.code)
})
stream.on('end', function () {
  console.log('Total records scanned: %d', recordCount)
})
```

The scan statement is initialized, and if:
- `concurrent: true` the scan queries nodes for records in parallel using threads; otherwise, sequentially queries each node. 
- `nobins: true` only returns record metadata (no bins). 
- `select:` only scans the specified record bins.

This example only returns the bin _baz_:

```js
var options = {
  concurrent: true,
  nobins: false,
  select: ['baz']
}
var scan = client.scan('test', 'demo', options)
```
