---
title: Error Handling
description: Error objects with the Aerospike Node.js client and the Aerospike database.
---

Each database operation returns an error object in the callback function:
- On success, `null` will be returned
- On error, the error object is populated. 

### Error Object

Error objects are of class `AerospikeError` and include the following properties:

- `code` &mdash; The operation status.
- `message` &mdash; The message corresponding to an error code. Can be an empty string if the operation completed successfully.
- `func` &mdash; The function in which the error occurred. Can be NULL if not captured properly in the operation.
- `file` &mdash; The file in which the error occurred. Can be NULL if not captured properly in the operation.
- `line` &mdash; The line number in the file in which the error occurred. Can be 0 if not captured properly in the operation.

{{#note}}
For server-side error codes, refer to the `Aerospike.status` module in the [API documentation](https://www.aerospike.com/apidocs/nodejs/module-aerospike_status.html).
{{/note}}

This example handles an error from a `put()` operation:

```js
client.put(key, record, function (error) {
  if (error) {
    console.error('error: %s [%d]\n%s', error.message, error.code, error.stack)
    process.exit(-1)
  }
})
```
