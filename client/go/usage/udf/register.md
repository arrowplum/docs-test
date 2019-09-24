---
title: Register a UDF
description: Use the Aerospike Go client UDFs to extend the functionality and performance of the Aerospike database.
---

This page describes registering a UDF module asynchronously with the Aerospike server. UDFs are grouped in a module that needs to register with the server. 

To register a UDF using `Client.RegisterUDF()`:

```go
// To automatically read the a UDF source file
func (clnt *Client) RegisterUDFFromFile(policy *WritePolicy, 
	clientPath string, 
	serverPath string, 
	language Language) (*RegisterTask, error)

// Supply the UDF source
func (clnt *Client) RegisterUDF(policy *WritePolicy, 
	udfBody []byte, 
	serverPath string, 
	language Language) (*RegisterTask, error)
```

Where:

- `udfBody` &mdash; The UDF source.
- `clientPath` &mdash; The UDF module path on the client machine.
- `serverPath` &mdash; The UDF module to store in the cluster.
- `language` &mdash; The UDF module language.

This example registers the UDF module _example.lua_.

```go
task, err = client.RegisterUDFFromFile(nil, "udf/example.lua", "example.lua", Language.LUA)
```

The UDF module registers asynchronously, so the server may return before the UDF module is available on all nodes. You can make the application wait for the asynchronous task to complete.

To block the call until the registration task completes:

```go
// to block until the UDF is created
for err := range task.OnComplete(); err != nil {
	// deal with the error here
}
// task is completed successfully
```

Once registration completes, the UDFs are available to any Aerospike client.
