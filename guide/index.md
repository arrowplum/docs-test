---
title: Feature Guide Overview
description: The feature guide for the Aerospike database.
assets: /docs/guide/assets
---

These feature guides present details on the following topics:

<div style="float: right" >
{{#figure "" "Database Structure" size="large"}}
![]({{book.assets}}/as-storage-architecture-small.png)
{{/figure}}
</div>

 - Aerospike is a row-oriented database, where every _record_ (equivalent to
a row in a relational database) is uniquely identified by a key. The record's
key and its other metadata live in the primary index. The record's data lives in
the pre-defined storage device of the namespace it occupies. For a more detailed
description read the [Data Model](/docs/architecture/data-model.html) section of
the architecture overview. Aerospike supports **[key-value store](/docs/guide/kvs.html)**
and [document store](/docs/guide/data-types.html#complex-data-types-cdts-) models.
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
- Aerospike [bins](/docs/architecture/data-model.html#bins) can each hold a distinct
[scalar](/docs/guide/data-types.html#basic-data-types) or [complex](/docs/guide/data-types.html#complex-data-types-cdts-) **data type**.
- Aerospike allows value-based **[queries](/docs/guide/query.html)** using [secondary indexes](/docs/architecture/secondary-index.html), where string and integer bin values are indexed and searched using equality (string or numeric) or range (numeric) filters.
- **[User-Defined Functions (UDFs)](/docs/guide/udf.html)** extend the functionality and performance capabilities of the Aerospike Database engine.
- In Aerospike, the **[aggregations](/docs/guide/aggregation.html)** framework allows fast, flexible query operations. Similar to _MapReduce_ systems, aggregation emits results in a highly parallel fashion. 
- Aerospike supports storage, indexing and querying of **[Geospatial](/docs/guide/geospatial.html)** data expressed as GeoJSON.

