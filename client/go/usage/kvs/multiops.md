---
title: Multiple Ops on a Record
description: Use the Aerospike Go client to modify and read multiple records in a single transaction. 
---

Use the Aerospike Go client to modify and read multiple records in a single transaction (that is, _atomic_ modification).

The Aerospike Go client API can perform the following operations on a record.

Operation | Method Name | Description | Conditions
--- | --- | --- | ---
write | WriteOp | Write a value to a bin. |
read | ReadOp | Read the value of a bin. |
read_header | ReadHeaderOp | Read only the record metadata (for example, generation and time-to-live). |
add | AddOp | Add an integer to the existing value of a bin. | The existing value must be integer.
append | AppendOp | Append string to the existing value of a bin. | The existing value must be string.
prepend | PrependOp | Prepend string to the existing value of a bin. | The existing value must be string.
touch | TouchOp | Rewrite the same record. | Updates generation and time-to-live values.

{{#note}}
Operations are performed in user defined order.
{{/note}}

### Operation Specification

To specify operations on different bins in the same transaction, create the bins with the values to apply:

```go
key, _ := as.NewKey("test", "demoset", "opkey")

bin1 := as.NewBin("optintbin", 7)
bin2 := as.NewBin("optstringbin", "string value")

client.PutBins(wpolicy, key, bin1, bin2)

bin3 := as.NewBin(bin1.Name, 4)
bin4 := as.NewBin(bin2.Name, "new string")
``` 

Create the appropriate operations using the bins and supply them to the `Operate()` method:

```go
record, err := client.Operate(policy, key, as.AddOp(bin3), 
	as.PutOp(bin4), as.GetOp())
```

The result returns as `Record`.
