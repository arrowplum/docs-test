---
title: Write a Record
description: Use the Aerospike Go client library to write records to the Aerospike database.
---

Use the Aerospike Go client library to write records to the Aerospike database.

For writes, the Aerospike Go client library provides various forms of `Client.PutBins()`.

Note that if they do not already exist, the set and bins are automatically created. You don't need to define a schema for the database. Aerospike is schemaless.

{{#note}}
Aerospike requires that namespaces are explicitely defined in config files. If you specify a namespace not yet defined, an error returns. _test_ is the default namespace specified in the server configuration file.
{{/note}}

This example sets a policy to specify the transaction re-transmission policy, timeout interval, record time-to-live (TTL), and what to do if the record exists. Instantiate the policy or pass in a NULL policy to initialize client defaults.


```go
// Initialize policy.
policy := as.NewWritePolicy(0, 0)
policy.Timeout = 50 * time.Millisecond  // 50 millisecond timeout.
```

### Write A Single Value

Records are identified using a key. This example creates a key _mykey_ to store in namespace _test_ within set _myset_. Other data types such as integers or byte arrays are valid data types for key.

```go
// Write single value.
key, _ := as.NewKey("test", "myset", "mykey")
bin := as.NewBin("mybin", "myvalue")

client.PutBins(policy, key, bin)
```

### Writing Multiple Values

To update multiple bins in a record, create additional bins and pass them to `PutBins()`:

```go
// Write multiple values.
bin1 := as.NewBin("name", "John")
bin2 := as.NewBin("age", 25)

client.PutBins(policy, key, bin1, bin2)
```

### Deleting a Bin

To delete a bin, set the bin value to `nil`:

```go
bin1 = as.NewBin(binName1, nil) // Set bin value to nil to drop bin

client.PutBins(policy, key, bin1)
```

### Writing An Object

Note, when writing an object:

- Save struct directly to the database.
- Attribute names become bin names, and attribute values are saved as bin values.
- Arrays and maps  automatically convert.
- Embedded structs automatically convert, and are saved in the database as maps.
- Use tags to alias the attribute (bin name) or to prevent the struct field from being persisted.
- Results are not language dependent. The record can be retrieved by other clients in different languages.

**Example**

```go
    type SomeStruct struct {
      A    int            `as:"a"`  // alias the field to a
      Self *SomeStruct    `as:"-"`  // will not persist the field
    }

    type OtherStruct struct {
      i interface{}
      OtherObject *OtherStruct
    }

    obj := &OtherStruct {
      i: 15,
      OtherObject: OtherStruct {A: 18},
    }

    key, _ := as.NewKey("ns", "set", value)
    err := client.PutObject(nil, key, obj)
    // handle error here

    rObj := &OtherStruct{}
    err = client.GetObject(nil, key, rObj)
```

### Modifying Write Behavior

The following is the default behavior for `PutBins()`:

- Only sends the key hash  to the database; not key value.
- Creates the record if it doesn't already exist in the cluster.
- Updates bin values in the record if the bins exist.
- Adds bins if they do not exist.
- Manages all record bins.

Use the`WritePolicy` object to modify write behavior. Allowed policy changes are:

- Only write if the record does not exist:
  - change the `WritePolicy` to `CREATE_ONLY`
- Completely replace a record _only_ if it exists:
  - change the `WritePolicy` to `REPLACE_ONLY`

### Writing the Record Time-to-live (TTL)

To specify a 2-second TTL value for a record on a write:

```go
writePolicy := as.NewWritePolicy(0, 0)
writePolicy.Expiration = 2 // seconds
client.PutBins(writePolicy, key, bin)
```

Where,

- `writePolicy.Expiration = 0` &mdash; Use the default TTL specified on the server side on each record update.
- `writePolicy.Expiration = math.MaxUint32` &mdash; Never expire.

### Read-Modify-Write

The read-modify-write (or check-and-set) process is:

1. Read the record.
1. Modify the record at the application.
1. Set the write policy to `EXPECT_GEN_EQUAL`` (to set optimistic client-side locking).
1. Write the modified data with the previous-read generation value.

