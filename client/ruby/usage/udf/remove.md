---
title: Remove a UDF
description:  Asynchronously remove a UDF module from the Aerospike server.
---

User-defined Functions (UDFs) are grouped in a module you can remove from the server. 

To remove a UDF using `Client.remove_udf`:

```ruby
def remove_udf(udf_name, options={})
```

Where:

- `udf_name` is the UDF module name.

To asynchronously delete the *example.lua* UDF module:

```ruby
task = client.remove_udf("udf/example.lua")
```

To block the call until the UDF is removed:

```ruby
# to block until the UDF is removed
task.wait_till_completed
```
