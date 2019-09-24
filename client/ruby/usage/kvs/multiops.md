---
title: Multiple Ops on a Record
description: Use the Aerospike Ruby client APIs to modify and read multiple record bins in a single transaction. 
---

Use the Aerospike Ruby client APIs to modify and read multiple record bins in a single transaction. This *atomic* modification returns the result.

Use the Aerospike Ruby APIs to perform the following operations.

Operation | Method Name | Description | Conditions
--- | --- | --- | ---
`write` | Operation.write | Write a value to a bin. |
`read` | Operation.read | Read the value of a bin. |
`read_header` | Operation.read_header | Read only the record metadata (generation and time-to-live). |
`add` | Operation.add | Add an integer to the existing bin value. | Integer only.
`append` | Operation.append | append string to the existing bin value | String only.
`prepend` | Operation.prepend | Prepend string to the existing bin value. | String only.
`touch` | Operation.touch | Rewrite the same record. | Metadata (generation and time-to-live) updates.

{{#note}}
Operations are performed in user defined order.
{{/note}}

### Operation Specification

To specify operations on different bins in the same transaction:

- Create bins with values to apply:

```ruby
key = Key.new('test', "demoset", "opkey")

bin1 = Bin.new("opt_int_bin", 7)
bin2 = Bin.new("opt_string_bin", "string value")

client.put(key, [bin1, bin2])
``` 

- Create the appropriate operations to use the bins and supply them to `Operate()`:

```ruby
bin3 = Bin.new(bin1.name, 4)
bin4 = Bin.new(bin2.name, "new string")

record = client.operate(
	key, 
	Operation.add(bin3), 
	Operation.put(bin4), 
	Operation.get
	)
```

The result returns in the `Record` object.

