---
title: Multiple Ops on a Record
description: Use the Aerospike C# Client to perform multiple operations on a record with the Aerospike database.
---

Applications can perform separate modification operations on multiple bins in a record within a single transaction using the Aerospike C# client. This also allows modification of the bins of a record, which is read back to the client within the transaction (that is, it allows an application to perform an atomic modification that returns the result).

The following operations can be performed on a record:

Operation | Description | Conditions
--- | --- | ---
WRITE | Write a value to a bin. |
READ | Read the value of a bin. |
READ_HEADER | Read only the metadata (generation and time-to-live) of the record. |
ADD | Add an integer to the existing value of a bin. | Only integer values.
APPEND | Append string to the existing value of a bin. | Only string values.
PREPEND | Prepend string to the existing value of a bin. | Only string values.
TOUCH | Rewrite the same record | Update the generation and time-to-live metadata.

{{#note}}
Operations are performed in user defined order.
{{/note}}

### Operation Specification

To specify operations on different bins for the same transaction:

- Create the bins with the values to apply:

```cs
Key key = new Key("test", "demoset", "opkey");
Bin bin1 = new Bin("optintbin", 7);
Bin bin2 = new Bin("optstringbin", "string value");
client.Put(policy, key, bin1, bin2);

Bin bin3 = new Bin(bin1.name, 4);
Bin bin4 = new Bin(bin2.name, "new string");
``` 

- Create the appropriate operations to use the bins, and supply them to the `Operate()` function:

```cs
Record record = client.Operate(policy, key, Operation.Add(bin3), 
    Operation.Put(bin4), Operation.Get());
```

The result returns in `record`.

