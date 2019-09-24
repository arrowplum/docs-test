---
title: Return the List of All UDFs on the Server
description: Use the Aerospike Ruby client to manage a list of UDFs.
---

Use the Aerospike Ruby client to manage a list of UDFs.

To get a list of registered UDFs on the server: 

```ruby
client.list_udf
```

An array of `Aerospike::UDF` objects returns with information about each UDF.
