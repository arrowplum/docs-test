---
title: Scanning Records
description: Use the Aerospike PHP client to scan all records in a specified namespace and set.
---

Use the Aerospike PHP client to scan all records in a specified namespace and set, and return to the client a stream of all records. Each record in the stream passes to a callback function for processing.

To halt record streams initiated by a scan or query, have the callback function return `false`.

### Creating a Scan

The [scan](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_scan.md) method includes namespace and set parameters to identify the set to scan, a callback record handler, and projects a subset of named bins.

```php
$db->scan("infosphere", "characters", function ($record) {
    var_dump($record); });
```

### Projecting Bins

This example only returns records containing the _name_ bin:

```php
$db->scan("infosphere", "characters", function ($record) {
    var_dump($record);
}, ["name"]);
```

### Halting the Stream

This example uses a callback function to stop the record stream after the third record is handled:

```php
$processed = 0;
$status = $db->scan("infosphere", "facts", function ($record) use (&$processed) {
    if (!is_null($record['bins']['quote'])) echo $record['bins']['quote']."\n";
    if ($processed++ > 3) return false; // halt the record stream
}, ["quote"]);
// check the status of the last operation
if ($status == Aerospike::ERR_SCAN_ABORTED) {
    echo "I think a sample of $processed records is enough\n";
} else if ($status !== Aerospike::OK) {
    echo "An error occured while scanning[{$db->errorno()}] {$db->error()}\n";
}
```

```
Beavers mate for life.
11 > 4
For Quality Carpets, Visit Kaplan's Carpet Warehouse
I think a sample of 3 records is enough
```

