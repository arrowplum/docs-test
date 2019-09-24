---
title: Register a UDF
description: Register a module containing a UDF asynchronously with the Aerospike server.
---

To register UDFs, you must first group them in a module and then register that UDF module with the Aerospike server cluster using `Client.register_udf`:

```ruby
# To automatically read the a UDF source file
def register_udf_from_file(client_path, server_path, language=Aerospike::Language::LUA, options={})

# Supply the UDF source as a string
def register_udf(udf_body, server_path, language=Aerospike::Language::LUA, options={})
```

Where:

- `udf_body` &mdash; The UDF source.
- `client_path` &mdash; The local path to the UDF module.
- `server_path` &mdash; The UDF module to store in the cluster.
- `language` &mdash; The UDF module language.

This example registers the _example.lua_ UDF module as a LUA language module:

```ruby
task = client.register_udf_from_file("udf/example.lua", "example.lua")
```

UDF modules register asynchronously, which means that the server can return before the UDF module is available on all nodes. The client can use `wait_till_completed` to wait until the asynchronous task completes:

```ruby
# to block until the UDF is created
task.wait_till_completed
# task is completed successfully
```

Once registration completes, the UDFs can be called from the Aerospike Ruby client.
