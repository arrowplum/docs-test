---
title: Managing Secondary Indexes
description: Use the Aerospike PHP client to create and destroy secondary indexes.
---

Use the Aerospike PHP client to create and destroy secondary indexes.

The set primary index contains a unique hash digest of its records used for key-value store operations. Create secondary indexes to query a specific record bin.

These admin methods manage secondary indexes:

- [createIndex()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_addindex.md)
- [dropIndex()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_dropindex.md)

```php
$status = $db->createIndex("test", "user", "email", Aerospike::INDEX_TYPE_STRING, "user_email_idx");
if ($status === Aerospike::OK) {
    echo "Index user_email_idx created on test.user.email\n";
else if ($status == Aerospike::ERR_INDEX_FOUND) {
    echo "This index has already been created.\n";
} else {
    echo "[{$db->errorno()}] ".$db->error();
}

// create another index on a different bin of the set
$db->createIndex("test", "user", "age", Aerospike::INDEX_TYPE_INTEGER, "user_age_idx");
```

