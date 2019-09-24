---
title: Examples
description: These source code examples demonstrate using the Aerospike Node.js client with the Aerospike database.
---

These source code examples demonstrate using the Aerospike Node.js client with the Aerospike database.

{{#note}}
`sudo` for Node Package Manager (npm) may or may not be required depending on your node installation.
{{/note}}

To run and build examples in the _examples_ directory:

```bash
$ cd examples
$ npm install ../
$ npm update
```

To independently run each example:

```bash
$ node <example>
```

Most examples require a key.

Use the `--help` flag to view usage details:

```bash
$ node <example> --help
```

The following are the included examples.

**Basic Operations**

- `exists.js` &mdash; Check the existence of a record.
- `get.js` &mdash; Read a record.
- `select.js` &mdash; Read specific bins of a record.
- `put.js` &mdash; Write a record.
- `remove.js` &mdash; Remove a record.
- `operate.js` &mdash; Perform multiple operations on a record.
- `info.js` &mdash; Get cluster status.

**Batch Operations**

- `batch_exists.js` &mdash; Check the existence of a set of records.
- `batch_get.js` &mdash; Read a batch of records.

**Range Operations**

- `range_get` &mdash; Read a range of records.
- `range_exists` &mdash; Check the existence of a range of records.
- `range_put` &mdash; Write a range of records.
- `range_remove` &mdash; Remove a range of records.
- `range_validate` &mdash; Write and validate a range of records.

