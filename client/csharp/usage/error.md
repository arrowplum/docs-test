---
title: Error Handling
description: Errors during an Aerospike databse server or client-side operation displays a message with the exact cause.
---

If an error occurs as a result of a database operation, an `AerospikeException` is thrown.

The `AerospikeException` `resultCode` field describes the exact cause of the error given by the Aerospike server.

Other client-side error conditions are:
- `AerospikeException.QueryTerminated` &mdash; Query prematurely terminated.
- `AerospikeException.Connection` &mdash; Client can't connect to the server.
- `AerospikeException.Timeout` &mdash; Database request expired before completing.
