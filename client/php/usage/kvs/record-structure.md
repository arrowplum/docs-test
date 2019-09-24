---
title: Record Structure
description: The Aerospike PHP client record structure.
---

[Aerospike stores data](/docs/architecture/data-model.html) in namespaces (_databases_ in  RDBMS) that contain sets (_tables_) of records uniquely identified using a key hash (20-byte RIPEMD-160) digest. An in-memory primary index over these digest values allows fast access to records.

Records comprise one or more bins, which are named, strongly-typed containers that contain both atomic (string, integer, bytes) and complex (map, list) data types. Records also contain these metadata values to track version and expiration or *time-to-live* (TTL):

- `generation` &mdash; The number of record modifications.
- `ttl`&mdash; The seconds remaining until the record expires ( default = 0; never expire). 
  - `ttl` resets on record writes or `touch()`.
  - Expired records are collected by the server. 

### Structure

Records contain:
- key: namespace, set, digest, key
- metadata: generation, ttl
- bins: _key-value pairs of data_

For example:

```bash
array(3) {
  ["key"]=>
  array(4) {
    ["digest"]=> 
    string(40) "436??b?0%c#c?"
    ["namespace"]=>
    string(10) "infosphere"
    ["set"]=>
    string(10) "characters"
    ["key"]=>
    NULL
  }
  ["metadata"]=>
  array(2) {
    ["generation"]=>
    int(2)
    ["ttl"]=>
    int(1337)
  }
  ["bins"]=>
  array(2) {
    ["name"]=>
    string(13) "Philip J. Fry"
    ["jobs"]=>
    array(4) {
      [0]=>
      string(19) "Delivery Boy (1999)"
      [1]=>
      string(19) "Delivery Boy (3000)"
      [2]=>
      string(5) "Pilot"
      [3]=>
      string(14) "The Mighty One"
    }
  }
}
```
{{#note}}
In this example, `$record["key"]["digest"]` is present and `$record["key"]["key"]` is NULL. `digest` is the unique identifier of the record, and the string or integer input hashed to produce it may or may not be stored with the record, depending on the option [Aerospike::OPT\_POLICY\_KEY](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike.md).
{{/note}}

