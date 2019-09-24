---
title: Data Management
description: Managing big data with volume and variety maintaining superior speed is the core of Aerospike.
assets: /docs/architecture/assets
---

Aerospike supports enhanced key-value operations. Data is structured in *bins* (RDBMS *columns*), each bin supports certain data types: integer, string, float, list, map, geojson, binary objects, or language-serialized objects.

Data management includes:

- Key-value operations, including in-database operations such as increment/decrement, and list and map element operations
- Data replication for high availability
- Automatic Data Expiration and Eviction
- Seamless upgrades and cluster size changes
- Flash (SSD) optimization
- Cross-datacenter replication (XDR)

Aerospike also supports:

- **[Complex data types](/docs/guide/data-types.html)** in bins (list and map, which can nest).
- **[Queries](/docs/guide/query.html)** use index strings and numeric bin values and the database searched by equality (string or numeric) or range (numeric).
- **[User-Defined Functions (UDFs)](/docs/guide/udf.html)** allow extended database processing by executing application code in Aerospike.
- **[Aggregations](/docs/guide/aggregation.html)** are a collection of records on which UDFs and aggregate values return.

