---
title: Policies
description: Use the Aerospike Node.js client to create policies for applications for use with the Aerospike database. 
---

Use the Aerospike Node.js client to create policies for applications for use with the Aerospike database.

### Policy Objects

Each operation accepts a policy object, which is a JavaScript object with
specified policy attributes. Attributes are optional. If no attributes are
specified, default values are used.

This example read policy specifies a timeout attribute for the `get()`
operation:

```js
client.get(key, {totalTimeout: 50}, function (error, record) {
    //process the result
})
```
 
#### Base Policy

All transaction policies support the following three policy values:

- `maxRetries` - Maximum number of retries before aborting the current transaction.
- `socketTimeout` - Socket idle timeout in milliseconds when processing a database command.
- `totalTimeout` - Total transaction timeout in milliseconds.

#### Write Policy

Write (`put`) operation policy supports the following properties:

- `commitLevel` - Specifies the number of replicas required to be committed successfully when writing before returning transaction succeeded.
- `compressionThreshold` - Minimum record size beyond which it is compressed and sent to the server.
- `durableDelete` - Specifies whether a tombstone should be written in place of a record that gets deleted as a result of this operation.
- `exists` – Specifies the behavior for the existence of the record (see [exists Attribute](#exists-Attribute)).
- `gen` – Specifies the behavior for the generation value (see [gen Attribute](#gen-Attribute)).
- `key` – Specifies the behavior for the key (see [key Attribute](#key-Attribute)).

#### Read Policy

Read (`get`/`select`) operation policy objects accept the following attributes:

- `consistencyLevel` - Specifies the number of replicas consulted when reading for the desired consistency guarantee.
- `key` – Specifies the behavior for the key (see [key Attribute](#key-Attribute)).
- `replica` - Specifies the replica to be consulted for the read operation.

#### Batch Policy

Batch (`batchRead`/`batchGet`) operations policy objects accept the following attributes:

- `allowInline` - Allow batch to be processed immediately in the server's receiving thread when the server deems it to be appropriate. If false, the batch will always be processed in separate transaction threads.
- `consistencyLevel` - Specifies the number of replicas consulted when reading for the desired consistency guarantee.
- `sendSetName` - Send set name field to server for every key in the batch. This is only necessary when authentication is enabled and security roles are defined on a per-set basis.

#### Remove Policy

Remove operations policy objects accept the following attributes:

- `commitLevel` - Specifies the number of replicas required to be committed successfully when writing before returning transaction succeeded.
- `durableDelete` - Specifies whether a tombstone should be written in place of a record that gets deleted as a result of this operation.
- `gen` – Specifies the behavior for the generation value (see [gen Attribute](#gen-Attribute)).
- `generation` – The generation of the record to remove.
- `key` – Specifies the behavior for the key (see [key Attribute](#key-Attribute)).

#### Info Policy

Info operations policy objects accept the following attributes:

- `checkBounds` – Ensure the request is within the size limit.
- `sendAsIs` – Send request without further processing

#### Operate Policy

Operate operations policy objects accept the following attributes:

- `commitLevel` - Specifies the number of replicas required to be committed successfully when writing before returning transaction succeeded.
- `consistencyLevel` - Specifies the number of replicas consulted when reading for the desired consistency guarantee.
- `durableDelete` - Specifies whether a tombstone should be written in place of a record that gets deleted as a result of this operation.
- `gen` – Specifies the behavior for the generation value (see [gen Attribute](#gen-Attribute)).
- `key` – Specifies the behavior for the key (see [key Attribute](#key-Attribute)).
- `replica` - Specifies the replica to be consulted for the read operation.

### Policy Attributes

This section describes the policy attributes.

#### key Attribute

How to use the key with the operation. 

The following valid attribute values for `key` are defined in *Aerospike.policy.key*:

- `DIGEST` – Calculate (on the client) and send the digest value of `key` to the server. We recommend using this mode of operation.
- `SEND` – Send the key. This attribute is ideal to reduce the number of bytes sent over the network, but only works when the combined `set` and `key` values are under 20 bytes. Use `policy.key.digest` for larger values.

**Example**

```js
let policy = new Aerospike.WritePolicy({
  key: Aerospike.policy.key.SEND
})
```

#### gen Attribute

Operation behavior based on the record generation value.

 The following valid attribute values for `gen` are defined in *aerospike.policy.gen*:

- `IGNORE` – Write a record, regardless of generation.
- `EQ` – Write a record, ONLY if generations are equal.
- `GT` – Write a record, ONLY if local generation is greater than remote.

**Example**

```js
let policy = new Aerospike.WritePolicy({
  gen: aerospike.policy.gen.GT
})
```

#### exists Attribute

Operation behavior based on the existence of the record.

The following valid attribute values for `exists` are defined in *aerospike.policy.exists*

- `IGNORE` – Write the record ignoring if it exists.
- `CREATE` – Only create a record if it doesn't exist.

**Example**

```js
let policy = new Aerospike.WritePolicy({
  exists: aerospike.policy.exists.CREATE
})
```
