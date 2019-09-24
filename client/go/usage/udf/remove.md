---
title: Remove a UDF
description: Use the Aerospike Go client UDFs to extend the functionality and performance of the Aerospike database.
---

User-Defined Functions (UDFs) are grouped in a module that you can remove from the server.

To remove a UDF using `Client.RemoveUDF()`:

```go
func (clnt *Client) RemoveUDF(policy *WritePolicy, udfName string) (*RemoveTask, error) {
```

Where:

- `udfName` &mdash; The UDF module name.

To delete the UDF module _example.lua_:

```go
task, err = client.RemoveUDF(nil, "udf/example.lua")
```

The UDF module is deleted asynchronously. 

To block the call until the UDF is removed:

```go
// to block until the UDF is removed
for err := range task.OnComplete(); err != nil {
    // deal with the error here
}
```

