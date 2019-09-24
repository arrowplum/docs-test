---
title: Register UDF
description: Use the Aerospike C client to manage UDFs with the Aerospike database.
---

Use the Aerospike C client to register, update, or remove a UDF module with the database. 

- `aerospike_udf_put()` — Register or Update a UDF module.
- `aerospike_udf_remove()` — Remove a UDF module.

Currently, the only UDF language Aerospike supports is Lua.

The following examples are excerpts from _examples/basic\_examples/udf_ in the Aerospike C client package.

To continue, the client must have an [established cluster connection](/docs/client/c/usage/connect).

## Reading the UDF


This example reads the file content to a local buffer, and returns it as an `as_bytes` object:

```cpp
FILE* file = fopen("myudf.lua", "r");

if (! file) {
	LOG("cannot open script file %s : %s", udf_file_path, strerror(errno));
	return false;
}

uint8_t* content = (uint8_t*)malloc(1024 * 1024);

if (! content) {
	LOG("script content allocation failed");
	return false;
}

uint8_t* p_write = content;
int read = (int)fread(p_write, 1, 512, file);
int size = 0;

while (read) {
	size += read;
	p_write += read;
	read = (int)fread(p_write, 1, 512, file);
}

fclose(file);

// Wrap the local buffer as an as_bytes object.
as_bytes udf_content;
as_bytes_init_wrap(&udf_content, content, size, true);

```

### Registering the UDF Module

To register the UDF module in the database cluster using `aerospike_udf_put()`:

```cpp
as_error err;
	

if (aerospike_udf_put(&as, &err, NULL, "myudf", AS_UDF_TYPE_LUA,
		&udf_content) != AEROSPIKE_OK) {
	LOG("aerospike_udf_put() returned %d - %s", err.code, err.message);
}

// Free the local buffer.
as_bytes_destroy(&udf_content);
```

`aerospike_udf_put()` sends the UDF module to a cluster node, which propagates it to the remaining nodes. UDF registration with all nodes in the cluster normally takes a few seconds.

To update UDF functionality, you must register the new version again, using the same module name.

### Checking UDF Module Registration

Use [Aerospike aql](/docs/tools/aql/udf_management.html) to confirm UDF module registration.

### Removing UDF Modules

To remove the UDF module from the server cluster using `aerospike_udf_remove()`:

```cpp
as_error err;
if (aerospike_udf_remove(&as, &err, NULL, "myudf") != AEROSPIKE_OK) {
	LOG("aerospike_udf_remove() returned %d - %s", err.code, err.message);
	return false;
}
```

