---
title: Write a Record
description: Use  the Aerospike Java client library to easily write records to the Aerospike Database.
---

The Aerospike Java client library provides various forms of `AerospikeClient.put()` for writing records in the database. Note that if they do not already exist, set and bins are automatically created.  You don't have to define a schema for the database.

This example sets one bin _mybin_ in the record _mykey_ with a string value _myvalue_. This example uses the Namespace _test_, which is the default namespace specified in the server configuration file.

```java
// Initialize policy.
WritePolicy policy = new WritePolicy();
policy.timeout = 50;  // 50 millisecond timeout.
```

For the transaction, the policy specifies the re-transmission policy, timeout interval, record expiration, and what to do if the record already exists. Instantiating the policy initializes client defaults. You can also pass a null policy to use client defaults.

### Writing Single Value

When writing records, you must identify the record in the database using a key. 

The following example creates a key for the record to insert. The value used for the _key_ is the string _mykey_, to be stored in the Namespace _test_ in the set _myset_. You can use other data types, such as integer for _key_.

```java
// Write single value.
Key key = new Key("test", "myset", "mykey");
Bin bin = new Bin("mybin", "myvalue");
client.put(policy, key, bin);
```

### Writing Multiple Values

To update multiple bins in a record, create additional bins and add them to the `put()` call:

```java
// Write multiple values.
Bin bin1 = new Bin("name", "John");
Bin bin2 = new Bin("age", 25);
client.put(policy, key, bin1, bin2);
```

### Deleting a Bin

To delete a bin, set the bin value to NULL:

```java
Bin bin1 = Bin.asNull(binName1); // Set bin value to null to drop bin.
client.put(policy, key, bin1);
```

### Modifying Write Behavior

The default behavior of a `put()` operation is:

- Create the record (if it doesn't exist) in the cluster.
- Add the bins if they do not exist.
- Update bin values in the record (if the bins already exist). 
- If the record has other bins, keep them in the record.

Use the `WritePolicy` object to change write behavior. Some policy changes are:

- Only write if the record does not already exist: 
	- Change the `WritePolicy` to `CREATE_ONLY`
- Completely replace a record _ONLY_ if it exists: 
	- Change the `WritePolicy` to `REPLACE_ONLY`

### Writing Record with Time-to-live (TTL)

This example specifies a time-to-live value for a record on a write, setting the record expiration to two seconds:

```java
WritePolicy writePolicy = new WritePolicy();
writePolicy.expiration = 2;
client.put(writePolicy, key, bin);
```

Set `writePolicy.expiration` to 0 to apply the default TTL on the server side each time a record updates.
Set `writePolicy.expiration` to -1 to specify that the record never expires.
 
### Read-Modify-Write

Record Read-Modify-Write (or Check-and-Set) involves:

- Reading the record.
- Modifying the record at the application level.
- Setting the write policy to `EXPECT_GEN_EQUAL`.
- Writing the modified data with the generation-previous read.
