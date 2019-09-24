---
title: Scans
description: Scan entire data sets in the Aerospike database without affecting baseline database transactions.
assets: /docs/guide/assets
---

Aerospike provides the functionality of enumerating and reading all data in a namespace. Applications can scan entire data sets in the Aerospike database without affecting baseline database transactions.

Use scans to:

- retrieve all (or a specified fraction of) records in a namespace or set (read-only scan).
- filter for records that are updates since a specific last-update-time (Aerospike version 3.12 and above)
- perform regular database maintenance by scanning all records in a set or namespace, and selectively updating records using a UDF (read-write scan).

Applications can send scan requests to all nodes in the cluster or to only one node. Within a node, scans work on all master partitions in a cluster node, one partition at a time.

### Read-Only Scan

In read-only scans, the scan initiates a request to each node in the cluster&mdash;in parallel&mdash;by executing a command from the client. As the scan iterates through each partition, it returns the current version of each record to the client.

Some read-only scan variations are:

- Scan only for records belonging to a set.
- Scan and return only record digests and metadata (generation and TTL).
- Scan and return specified bins.
- Scan and return only a (randomly sampled) specified fraction of the records.
- Scan and return filtered records with last-update-time > X. See [Predicate Filter](/docs/guide/predicate.html).

Many database tasks such as index creation and backups also use data scan as the underlying implementation mechanism.

During this process, a client program can also write data.

### Read-Write Scan

Clients can also scan the database and apply a Lua user-defined function (UDF) to each record. this is more efficient than the client-side scan for cases where data needs to be manipulated.

Use read-write scans for database maintance, and can provide arbitrary rules for grooming your data. For example, use a UDF to compare the `last_visited` value of a record. If the value is too old (implying that it was not touched for a long time), the application can delete the record. The application can apply a combination of rules such as if the `last_visited` value is old and the customer is non-paying, but not a personal friend. 

The application can also use generic grooming functions and pass parameters when the scan executes. This approach is quite powerful because cleanup processing is done as close to the data source as possible.

Such UDF scans run in the background and do not return any data to the client. They are commonly executed using adminstrator tools, and do not require client programming.

### Version 3.6.0 Enhancements

In version 3.6.0 and later, scans have the following enhancements:
- Concurrent scans are interlaced so that all scans can simultaneously progress.
- Only `scan-max-active` number of scans can be active at anytime. New scans are rejected and the `AS_PROTO_RESULT_FAIL_FORBIDDEN` error returns.
- For read-only scans, increasing the number of threads dedicated to  can subsystem to achieve more parallelism. Use this method with caution and monitor system performance.
- For scans that execute a UDF on each record (read-write scans), the [`scan-max-udf-transactions`](/docs/reference/configuration/#scan-max-udf-transactions) parameter controls parallelism and how many records the scan can concurrently process.
- Expedite active scans expedited by changing their priority. Higher priority scans interrupt lower priority scans until all higher priority scans complete.

See [Manage Scan](/docs/operations/manage/scans) for more information.

### Known Limitations

- During cluster state changes, a pending scan can result in the return of duplicate records or a partial number of records.

### References

See these topics for language-specific examples:

- [Java](/docs/client/java/usage/scan/scan.html)
- [C# .NET](/docs/client/csharp/usage/scan/scan.html)
- [Node.js](/docs/client/nodejs/usage/scan/scan.html)
- [Go](/docs/client/go/usage/scan/scan.html)
- [Python](/docs/client/python/usage/scan/scan.html)
- [PHP](/docs/client/php/usage/scan/scan.html)
- [Ruby](/docs/client/ruby/usage/scan)
- [C](/docs/client/c/usage/scan/scan.html)
