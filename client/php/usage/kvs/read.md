---
title: Reading Records
description: Use the Aerospike PHP client record read methods to get data from the Aerospike database. 
---

These methods get data from the Aerospike database:

- <a href="https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_get.md" target="_api">get()</a> &mdash; Reads specified bins in a record.
- <a href="https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_exists.md" target="_api">exists()</a> &mdash; Checks whether record metadata exists.
- <a href="https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_getmany.md" target="_api">getMany()</a> &mdash; Reads a batch of records.
- <a href="https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_existsmany.md" target="_api">existsMany()</a> &mdash; Retrieve metadata for a batch records.

### Defining the Key

To define the tuple (`namespace`, `set`, `key`) that uniquely identify a record:

```php
$key = $db->initKey("infosphere", "characters", 1);
```

### Reading a Record

This option gets the `key` component stored with the record on the previous write, applies a filter, and gets the record using the digest. If no key component is found, only `["key"]["digest"]` is filled.

```php
$option = [Aerospike::OPT_POLICY_KEY => Aerospike::POLICY_KEY_SEND];
$status = $db->get($key, $record, [], $option);
if ($status == Aerospike::ERR_RECORD_NOT_FOUND) {
    echo "A character with key ". $key['key']. " does not exist in the database\n";
    exit(2);
}

// filter the record down to a few specific bins
$status = $db->get($key, $record, ["name", "jobs"]);
if ($status == Aerospike::ERR_RECORD_NOT_FOUND) {
    var_dump($record);
}

// get the record using the digest value
$key_digest = $db->initKey("infosphere", "characters", $record["key"]["digest"], true);
$db->get($key_digest, $again);
```

### Checking Record Existence

This example checks record existence by attempting to retrieve its metadata. 

{{#note}}
`exists()` is a faster operation than `get()` because it only scans the record <a href="/docs/architecture/primary-index.html" target="architecture">Primary Index</a>. 
{{/note}}

```php
$status = $db->exists($key, $metadata);
if ($status == Aerospike::OK) {
    var_dump($metadata);
}
```

Example return:

```bash
array(2) {
  ["generation"]=>
  int(2)
  ["ttl"]=>
  int(1337)
}
```

### Batch Operations

Use `getMany()` and `existsMany()` for multiple key accessment.

```php
$keys = [];
for ($i = 1; $i <= 4; $i++) {
  $keys[] = $db->initKey('infosphere', 'characters', $i);
}
var_dump($db->existsMany($keys));
var_dump($db->getMany($keys));
```

