---
title: Read a Record
description: Use the Aerospike Go client APIs to read records from the Aerospike database.
---

Use the Aerospike Go client APIs to read records from the Aerospike database.

To read a record from the database, the Aerospike Go client library provides the following forms of `Client.Get()`:

- Read all bins in a record.
- Read only specified bins in a record.
- Read only the record metadata.
- Check the existence of a record.

Each method returns `Record`, which contains the record metadata and bins.

See [Batch Reads](batch.html) to read multiple records within the same call.

### Reading All Bins of a Record

To read all bins of a record, use `Client.Get()`. This reads the record and metadata from the database and returns a `Record` object.

```go
record, err := client.Get(policy, key)
```

### Reading Specific Bins of a Record

To read specific bins of a record, use `Client.Get()` and specify each bin to return. This reads the record and metadata from the database and returns a `Record` object.

This example reads the _name_ and _age_ bins from the record:

```go
record, err := client.Get(policy, key, "name", "age")
```

### Checking Record Existence

To check if a record exists using `Client.Exists()`:

```go
exists, err := client.Exists(policy, key)
```

You can also use batch requests. The resulting array has the same order as the key array:

```go
keys := make(*Key, size)

for i := 0; i < size; i++ {
	keys[i], _ = as.NewKey("test", "myset", (i + 1))
}

existsArray, err := client.BatchExists(policy, keys)
```

### Writing and Reading An Object

To read a record and write bin values into a struct (attribute names are determined by the bin names):

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

