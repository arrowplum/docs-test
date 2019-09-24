---
title: Error Handling
description: On Aerospike database server or client-side operation errors, the return value describes the exact cause.
categories:
  - aerospike-client-ruby
tags:
  - aerospike-client-ruby
  - ruby
---

If an error occurs as a result of the database operation, an exception is raised. `Aerospike::Exceptions::Aerospike` is inherited from `StandardError`, and can be caught using normal `rescue` clauses.

The `Aerospike::Exceptions::Aerospike` class provides an accessor to `result_code` that describes the exact cause of the Aerospike server error.

{{#note}}
Client-side internal exceptions, either originating from the client code or from the Ruby standard library, are not converted to an `Aerospike` exception.
{{/note}}
