---
title: Introduction - PHP Client
description: Use the Aerospike PHP client to build PHP applications to store and retrieve data from an Aerospike cluster.
---

Use the Aerospike PHP client to build PHP applications to store and retrieve data from an Aerospike cluster.

The Aerospike PHP client is built on top of the Aerospike C client, as an extension for PHP versions >= 7.0.

### Code

This example connects to the Aerospike cluster, creates a key, the records and their bins, and reads the bin values.

```php
// The cluster can be located by just one host
$config = [
  "hosts" => [
    [ "addr" => "127.0.0.1", "port" => 3000 ]]];

// The new client will connect and learn the cluster layout
$db = new Aerospike($config);

// records are identified as a (namespace, set, primary-key) tuple
$key = $db->initKey("infosphere", "characters", 1234);

// the record and its bins are created and updated similar to an array
$put_vals = [ "name" => "Scruffy", "Occupation" => "The Janitor" ];
$db->put($key, $put_vals);

// record bins can hold complex types such as arrays
$update_vals = [
  "quotes" => [
    "I'm Scruffy. The janitor.",
    "I'm on break.",
    "Scruffy's gonna die like he lived."]];
$db->put($key, $update_vals);

// read selected bins from a record
$db->get($key, $record, ["name", "quotes"]);

$db->close();
```

### Data Model

At the top of the data model is the **namespace**. This container has one set of policy rules for all data. Namespace is similar to the _database_ concept in an RDBMS, only it is distributed across the cluster. A namespace is divided into **sets**, similar to RDBMS _tables_.

Bins are pairs of key-value data contained  in **records**. These concepts are similar to _columns_ of a _row_ in RDBMS. Aerospike is schemaless. You don't have to define bins in advance.

Records are uniquely identified by their **key**, and sets have a primary index that contains the keys of all records in the set.

### Legacy PHP Client

If you are using PHP 5.x, we have a [legacy PHP client](https://github.com/aerospike/aerospike-client-php5) that supports PHP 5.3, 5.4, 5.5, and 5.6.

<div class="text-center">
<a class="button primary" href="/docs/client/php/start">GET STARTED</a>
</div>
