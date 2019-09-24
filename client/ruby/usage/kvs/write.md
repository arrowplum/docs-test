---
title: Write Records
description: Use the Aerospike Ruby client APIs to write records to the Aerospike database.
---

Use the Aerospike Ruby client `#put` APIs to write records to the Aerospike database.

The set and bins are created automatically if they do not exist. The Aerospike database is schemaless. There is no need to define a schema.

{{#note}} 
Aerospike requires that namespaces are explicitly defined in the config files. If the a database (namespace) isn't defined, an error returns.
{{/note}}

These examples set one bin *mybin* in record *mykey* with a string value *myvalue*. The namespace is *test*, which is the default namespace specified in the server configuration file.

### Writing a Single Value

On record writes, you must identify the record in the database using a key. 

This example creates the key *mykey* for the record to insert to store in namespace *test* within set *myset*. Other data types (such as integer) can be used for key.

```ruby
key = Key.new('test', 'myset', 'my key')
bin = Bin.new('mybin', 'my value')

# set timeout to 50 milliseconds
client.put(key, bin, :timeout => 0.05)
```

### Writing Multiple Values

To update multiple bins in a record, create additional bins and pass them as an array:

```ruby
bin1 = Bin.new('name', 'John')
bin2 = Bin.new('age', 25)

client.put(key, [bin1, bin2])
```

### Deleting a Bin

To delete a bin, set the bin value to `nil`:

```ruby
bin1 = Bin.new(binName1, nil) # Set bin value to nil to drop bin

client.put(key, bin1)
```

### Modifying Write Behavior

The default `#put` behavior is:

- Only send the key hash not the key value to the database.
- Create the record if it doesn't already exist in the cluster.
- Update bin values in the record if the bins already exist. 
- Add the bins if they do not exist.
- Keep any other bins in the record.

Change write behavior using the `WritePolicy` object. Example policy changes are:
- Only write if the record does not already exist:
  - change `WritePolicy` to `CREATE_ONLY`
- Replace a record *only* if it exists:
  - change `WritePolicy` to `REPLACE_ONLY`

### Writing Record Time-to-Live (TTL)

To specify a 2-second record expiration (time-to-live) value:

```ruby
client.put(key, bin, :expiration => 2)
```

- 0 = Apply TTL on update
- 1 = Record never expires
 
### Read-Modify-Write

Read-modify-write (or check-and-set) is a common record operation that:
- Reads the record.
- Modifies the record at the application level.
- Sets the write policy to `EXPECT_GEN_EQUAL` (for optimistic client-side locking).
- Write the modified data with the previous `generation`.

