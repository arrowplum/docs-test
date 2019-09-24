---
title: Return the List of All UDFs on the Server
description: Use the Aerospike Go client UDFs to extend the functionality and performance of the Aerospike database. 
---

Use this method to manage a list of UDFs.

To retrieve a list of registered UDFs on the server: 

```go
func (clnt *Client) ListUDF(policy *BasePolicy) ([]*UDF, error)
```

For example,

```go
udfList := client.ListUDF(policy)
```

This returns `[]*UDF` with information about each registered UDF.
