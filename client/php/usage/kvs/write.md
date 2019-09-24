---
title: Writing Records
description: Use the Aerospike PHP client put methods to write record data in the Aerospike database. 
---

Use the Aerospike PHP client put methods to write record data in the Aerospike database. The _Aerospike_ class includes several methods for writing records, and updating and overwriting record bins. These write methods act like PHP array operations.

The Aerospike database is schemaless. New record bins are created when inserted. Write operations overwrite bins unless the default policy is modified.

Aerospike [natively supports](/docs/guide/data-types.html) string, integer, byte (binary data), list (indexed array), and map (associative array) data types. To retain their type, other data types (such as boolean and object) are serialized on writes and de-serialized on reads.

The Aerospike PHP client write methods are:

- [put()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_put.md) &mdash; Writes a record at the specified key.
- [increment()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_increment.md) &mdash; Increments integer bin values.
- [append()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_append.md) &mdash; Adds a string to  string bins.
- [prepend()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_prepend.md) &mdash; Prepends a string to the value of a string type bin.
- [operate()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_operate.md) &mdash; Combines multiple bin operations into one operation.

### Writing a Record

In this example, `put()` follows the default policy `Aerospike::POLICY\_EXISTS\_IGNORE` (or `CREATE\_OR\_UPDATE`) which has array-like behavior. The client policies are strict, like SQL. Use client policies to separate `CREATE`, `UPDATE`, and `REPLACE` behavior.

```php
$key = $db->initKey("infosphere", "characters", 3);
$bins = ["name" => "Bender", "Occupation" => "Bender", "age" => 1055];
// store the key data with the record, rather than just its digest
$option = [Aerospike::OPT_POLICY_KEY => Aerospike::POLICY_KEY_SEND];
$db->put($key, $bins, 0, $option);
```

### Updating and Adding Bins

This example transforms _Occupation_ from a string to an array (to _list_ on the Aerospike server), and adds the bin _Alma Mater_ to the record.

```php
$bins = [
  "Occupation" => ["Bender", "Criminal", "Iron Chef"],
  "Alma Mater" => "Bending State University"];
$db->put($key, $bins);
```


In this example, the [Aerospike::OPT\_POLICY\_EXISTS](https://github.com/aerospike/aerospike-client-php/blob/mastIn thr/doc/aerospike.md) option sets other `Aerospike::POLICY\_EXISTS\_\*` values.

```php
// the following should only work to 'INSERT' a brand new record
$bins = ["fail" => "because the record already exists"];
$option = [Aerospike::OPT_POLICY_EXISTS => Aerospike::POLICY_EXISTS_CREATE];
$status = $db->put($key, $bins, 120, $option);
if ($status == Aerospike::ERR_RECORD_EXISTS) {
  echo "This would have worked if we used Aerospike::POLICY_EXISTS_UPDATE\n";
}
```

### Using Generation for Check-and-Set

Using the record `generation` metadata, subsequent write operations can implement a check-and-set (CAS) pattern to ensure that no other write occured since the last read. The [Aerospike::OPT\_POLICY\_GEN](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike.md) option passes with `put()`.

```php
$current_gen = $record["metadata"]["generation"];
$ttl = $record["metadata"]["ttl"];
// only write if nothing has changed since the last read
$option = [Aerospike::OPT_POLICY_GEN => [Aerospike::POLICY_GEN_EQ, $current_gen]];
$status = $db->put($key, ["foo" => "bar"], $ttl, $option);
if ($status == Aerospike::ERR_RECORD_GENERATION) {
  echo "The generation policy check failed\n";
}
```

### Appending and Prepending a Bin

```php
$db->prepend($key, "name", "Captain ");
$db->append($key, "Alma Mater", ", Santa Cruz");
```

### Multi-Ops

To combine multiple bin operations on a single record with optional post-write reads for the modified bin values:

```php
$operations = [
  ["op" => Aerospike::OPERATOR_WRITE,
   "bin" => "Occupation", "val" => ["Bender", "Criminal", "Iron Chef"]],
  ["op" => Aerospike::OPERATOR_INCR,
   "bin" => "age", "val" => -1],
  ["op" => Aerospike::OPERATOR_APPEND,
   "bin" => "name", "val" => ' "Bending" Rodriguez'],
  ["op" => Aerospike::OPERATOR_READ,
   "bin" => "age"]
];
$status = $db->operate($key, $operations, $returned);
if ($status == Aerospike::OK) {
  var_dump($returned); // display the age
}
```

