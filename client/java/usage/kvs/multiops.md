---
title: Multiple Ops on a Record
description: Use the Aerospike Java client API to modify and read back to the client bins of a record in a single transaction. 
---

You can use the Java client API to perform separate operations on multiple bins in a record in a single transaction. This feature allows the application to modify and read bins of a record in a single transaction (that is, perform an atomic modification that returns the result).

Some record operations include:

Operation | Description | Conditions
--- | --- | ---
write | Write a value to a bin. |
read | Read the value of a bin. |
read_header | Read only the metadata (generation and time-to-live) of the record. |
add | Add an integer to the existing value of the bin. | The existing value must be integer.
append | Append a string to the existing value of the bin. | The existing value must be string.
prepend | Prepend a string to the existing value of the bin. | The existing value must be string.
touch | Rewrite the same record. | Generation and time-to-live are updated.

{{#note}}
Operations are performed in user defined order.
{{/note}}

### Operation Specification

To specify operations on different bins in the same transaction, create the bins with the values to apply:

```java
Key key = new Key("test", "demoset", "opkey");
Bin bin1 = new Bin("optintbin", 7);
Bin bin2 = new Bin("optstringbin", "string value");	
client.put(policy, key, bin1, bin2);

Bin bin3 = new Bin(bin1.name, 4);
Bin bin4 = new Bin(bin2.name, "new string");
``` 

Create the appropriate operations using the bins and supply them to the `operate()` function:

```java
Record record = client.operate(policy, key, Operation.add(bin3), 
	Operation.put(bin4), Operation.get());
```
