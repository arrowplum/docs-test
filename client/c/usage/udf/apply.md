---
title: Applying a UDF on a Record
description: A user-defined function (UDF) executes on a single row (a Record UDF) to update or create a record.
---

Use the Aerospike C client API `aerospike_key_apply()` to apply a **Record UDF** against a record in the database.

Before using `aerospike_key_apply()`, the UDF module that contains the function must register with the Aerospike server. See [Register UDF](/docs/client/c/usage/udf/register.html).

This example is an excerpt from _examples/basic\_examples/get_ in the Aerospike C client package.

To continue, the client must have an [established the cluster connection](/docs/client/c/usage/connect).

### Defining the UDF

To define `bin_transform` in the `basice_udf` module:

```lua
function bin_transform(record, bin_name, x, y)
	record[bin_name] = (record[bin_name] * x) + y
	aerospike:update(record)
	return record[bin_name]
end
```

This simple arithmetic UDF has three arguments: _bin\_name_, _x_, and _y_. It updates the value in the specified bin after performing the arithmetic operation, and returns the resulting bin value.

### Initializing a Key

The application identifies records in the database using a key. In this example, the _key_ value is the string _test-key_, stored in namespace _test_ within set _test-set_. _key_ can be other data types such as integer or blob. 

```cpp
as_key key;
as_key_init_str(&key, "test", "test-set", "test-key");
```

### Passing Parameters to the UDF

The _bin\_transform_ UDF requires a string parameter and two integer parameters, so first, populate an argument list:

```cpp
as_arraylist args;
as_arraylist_inita(&args, 3);
as_arraylist_append_str(&args, "test-bin-2");
as_arraylist_append_int64(&args, 4);
as_arraylist_append_int64(&args, 400);
```

### Applying a UDF on a Record

To call the UDF on the record in the database using _key_:

```cpp
as_val * result = NULL;
if (aerospike_key_apply(&as, &err, NULL, &key, "mymodule", "add", 
	(as_list *) &args, &result) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

The value from the `bin_transform()` UDF returns in the `result` parameter:

- If _key_ does not exist, `AEROSPIKE_ERR_RECORD_NOT_FOUND` returns.
- If the UDF is found or if there is a run-time error executing the UDF, details on the error returns in `as_error`.

### Cleaning Up Resources

Use `as_val_destroy()` to release the UDF and associated resources:

```cpp
as_val_destroy(&result);
```

Use `as_arraylist_destroy()` to clean up the arguments list:

```cpp
as_arraylist_destroy(&args);
```

{{#note}}
The key does not require an explicit destroy call since it and its members are created on the stack.
{{/note}}

