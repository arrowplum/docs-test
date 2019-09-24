---
title: Examples
description: Aerospike PHP client API examples.
---

The Aerospike PHP client package includes [example scripts](https://github.com/aerospike/aerospike-client-php/tree/master/examples) that demonstrate the features of the PHP extension.

The examples have a similar interface viewed by calling the example with the `--help` flag. These examples access an Aerospike server with the IP address `192.168.119.3`.

There are more basic examples [here](https://github.com/aerospike/aerospike-client-php/tree/master/examples/basic_examples).

#### Record Operations

`rec-operations.php` has examples of record-level operations (such as `get()`, `remove()`, `touch()`, `exists()`, and `put()`) with various policy options set (`Aerospike::OPT\_POLICY\_GEN`, `Aerospike::OPT\_POLICY\_EXISTS`), and an example of record identification using its digest (the hash key on the server).

```bash
php rec-operations.php --host=192.168.119.3 -a -c
```

#### Batch Operations

`batch-operations.php` has example batch record operations `getMany()` and `existsMany()`.

```bash
php batch-operations.php --host=192.168.119.3 -a -c
```

#### Bin Operations

`bin-operations.php` has example record bin modification operations such as `prepend()`, `append()`, `increment()`, `operate()` (multi-ops), and `removeBin()`.

```bash
php bin-operations.php --host=192.168.119.3 -a -c
```

#### Applying a Record UDF

The example in `udf-operations.php` applies two Record user-defined functions (UDFs) to a record bin.

```bash
php udf-operations.php --host=192.168.119.3 -a -c
```

### Scan Method Examples

This section describes the scan method examples [here](https://github.com/aerospike/aerospike-client-php/tree/master/examples/scan_examples).

#### Simple Scan

The `standard.php` example invokes a callback method for each record streamed from the server as it scans the _users_ set in the _test_
namespace.

```bash
php standard.php --host=192.168.119.3 -a -c
```

#### Buffering Results With a Limit

`buffered.php` demonstrates buffering the records streaming back from a scan into a `$result` array, and stopping the scan by returning `false` at the specified expiration limit.

```bash
php buffered.php --host=192.168.119.3 -a -c
```

#### Scan Applying a UDF in the Background

`background.php`demonstrates applying a UDF to each record in a background scan of set _users_ in namespace _test_, transforming the record _age_ bin value, and displaying the records.

```bash
php background.php --host=192.168.119.3 -a -c
```

### Query Method Examples

This section describes the query examples [here](https://github.com/aerospike/aerospike-client-php/tree/master/examples/query_examples).

#### Query Example

`simple.php` invokes a callback method for each record streamed from the server as it queries the _characters_ set in the _test_
namespace, creates a secondary index on a bin in this set.

```bash
php simple.php --host=192.168.119.3 -a -c
```

#### Applying a Stream UDF to the Results of a Query

`aggregate.php` applies UDF aggregation (similar to SQL `COUNT(\*) GROUP BY _field_`) to records streaming out of a secondary index query.

```bash
php aggregate.php --host=192.168.119.3 -a -c
```

### Info Method Examples

This section describes the info examples [here](https://github.com/aerospike/aerospike-client-php/tree/master/examples/info_examples).

#### Basic Info Operations

`standard.php` demonstrates the `info()`, `infoMany()`, and `getNodes()` info methods.

```bash
php standard.php --host=192.168.119.3 -a
```

