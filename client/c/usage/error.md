---
title: Error Handling
description: Manage errors using the Aerospike C client and the Aerospike database.
---

Each database operation accepts an `as_error` object as an argument. When an error occurs during an operation, the `as_error` argument is populated with the status code and error information. The `as_error` argument is usually the second argument to a database operation.

The `as_error` object provides the following information:

- `code` &mdash; The `as_status` of the operation.
- `message` &mdash; The message that corresponds to an error code. If the operation completed successfully, this can be an empty string.
- `func` &mdash; The function in which the error occurred. If it was not captured properly in the operation, this can be be NULL.
- `file` &mdash; The file in which the error occurred. If it was not captured properly in the operation, this can be be NULL.
- `line` &mdash; The line number in the file where the error occurred. If it was not captured properly in the operation, this can be be 0.

When an operation completes, it returns an `as_status` that corresponds to the `as_error.code`. If an operation returns a failure status&mdash;anything not `AEROSPIKE_OK`&mdash;then the application should check the `as_error` variable for more information. See `as_status.h` for a complete list.

An `as_error` can be reused between operations, because each operation resets the `as_error` and sets on error.

The following example handles the error from an operation:

```cpp
if (aerospike_key_put(&as, &err, NULL, &key, &rec) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

The application can do more elaborate error handling.

