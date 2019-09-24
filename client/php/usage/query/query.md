---
title: Query Records
description: Use the Aerospike PHP client to query a set.
---

Use the Aerospike PHP client to perform queries on a [secondary index](/docs/guide/query.html#secondary-index) of a set. Similar to a scan, records matched to the query return to the client as a stream of records. Each record in the stream passes to a callback function for processing.

To halt record streams initiated by a scan or query, have the callback return `false`.

### Creating a Query

The [query()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_query.md) method includes namespace and set parameters to identify the set to query, a predicate, a callback record handler, and (optionally) a subset of named bins to project.

#### Predicate Helpers

The `$where` predicate argument must follow a strict structure, so the Aerodpike PHP client provides these helper methods:

- [predicateEquals()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_predicateequals.md)
- [predicateBetween()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_predicatebetween.md)

```php
$where = Aerospike::predicateBetween("age", 30, 39);
$status = $db->query("infosphere", "characters", $where, function ($record) {
    var_dump($record); });
```

### Projecting Bins

This example only returns records that include the _name_ bin:

```php
$where = Aerospike::predicateBetween("age", 30, 39);
$status = $db->query("infosphere", "characters", $where, function ($record) {
    var_dump($record);
}, ["name", "age"]);
```

### Buffering Records

This example buffers the records streaming back from the server into the `$result` variable:

```php
$result = array();
$where = Aerospike::predicateBetween("age", 30, 39);
$status = $db->query("test", "users", $where, function ($record) use (&$result) {
    $result[] = $record['bins'];
});
if ($status !== Aerospike::OK) {
    echo "An error occured while querying[{$db->errorno()}] {$db->error()}\n";
} else {
    echo "The query returned ".count($result)." records\n";
}
```

