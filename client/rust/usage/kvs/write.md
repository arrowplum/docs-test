---
title: Write a Record
description: Use  the Aerospike Rust client library to easily write records to the Aerospike Database.
---

The Aerospike Rust client's `put()` method writes records to the database.
Note that if they do not already exist, set and bins are automatically created.
You don't have to define a schema for the database.

This example sets one bin _mybin_ in the record _mykey_ with a string value
_myvalue_. This example uses the Namespace _test_, which is the default
namespace specified in the server configuration file.

```rust
let policy = WritePolicy::default();
policy.timeout = 50;  // 50 millisecond timeout.
```

For the transaction, the policy specifies the re-transmission policy, timeout
interval, record expiration, and what to do if the record already exists.

### Writing Single Value

When writing records, you must identify the record in the database using a key. 

The following example creates a key for the record to insert. The value used
for the _key_ is the string _mykey_, to be stored in the Namespace _test_ in
the set _myset_. You can use other data types, such as integer for _key_.

```rust
let key = as_key!("test", "myset", "mykey");
let bin = as_bin!("mybin", "myvalue");
match client.put(&policy, &key, &vec![&bin]) {
    Ok(()) => println!("Record written"),
    Err(err) => println!("Error writing record: {}", err),
}
```

### Writing Multiple Values

To update multiple bins in a record, create additional bins and add them to the `put()` call:

```rust
// Write multiple values.
let bin1 = as_bin!("name", "John");
let bin2 = as_bin!("age", 25);
client.put(&policy, &key, &vec![&bin1, &bin2]).unwrap();
```

### Deleting a Bin

To delete a bin, set the bin value to the empty value:

```rust
let bin1 = as_bin!("age"); // Set bin to empty value to drop the bin.
client.put(&policy, &key, &vec![&bin1]).unwrap();
```

### Modifying Write Behavior

The default behavior of a `put()` operation is:

- Create the record (if it doesn't exist) in the cluster.
- Add the bins if they do not exist.
- Update bin values in the record (if the bins already exist). 
- If the record has other bins, keep them in the record.

Use the `WritePolicy` to change write behavior. Some policy changes are:

- Only write if the record does not already exist: 
  - Set `policy.record_exists_action` to `RecordsExistAction::CreateOnly`.
- Completely replace a record _ONLY_ if it exists: 
  - Set `policy.record_exists_action` to `RecordsExistAction::ReplaceOnly`

### Writing Record with Time-to-live (TTL)

This example specifies a time-to-live value for a record on a write, setting the record expiration to two seconds:

```rust
let policy = WritePolicy::default();
policy.expiration = Expiration::Seconds(2);
client.put(&policy, &key, &bins).unwrap();
```

Set `policy.expiration` to `Expiration::NamespaceDefault` to apply the default TTL on the server side each time a record updates.
Set `policy.expiration` to `Expiration::Never` to specify that the record never expires.
Set `policy.expiration` to `Expiration::DontUpdate` to update a record without chaning it's TTL. (Requires Aerospike Server version 3.10.1 or later.)
 
### Read-Modify-Write

Record Read-Modify-Write (or Check-and-Set) involves:

- Reading the record.
- Modifying the record at the application level.
- Setting `policy.generation_policy` to `GenerationPolicy::ExpectGenEqual`
  while setting `policy.generation` to the previously set generation
  value.
