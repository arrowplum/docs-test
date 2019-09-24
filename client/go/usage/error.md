---
title: Error Handling
description: Errors during an Aerospike databse server or client-side operation displays a message with the exact cause.
categories:
  - aerospike-client-go
tags:
  - aerospike-client-go
  - go
---

If an error occurs as a result of the database operation, an `AerospikeError` returns.

The `AerospikeError` `ResultCode()` method describes the exact cause of the error given by the Aerospike server.

Errors can also occur internally on the client side, either originating from the client code or from within the _go stdlib_. These errors are not converted to an `AerospikeError` instance unless they are `io.Timeout` errors.
