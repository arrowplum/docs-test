---
title: Record UDF Example – Message Queue
description:  Record UDF Example – Message Queue
---

The example that used to live here has been removed, because the functionality
implemented in Lua is now native to the [List](/docs/guide/cdt-list.html) data
type.

Native operations perform and scale better than UDFs, and can be chained into
single record transactions. Whenever possible, they should be used over a UDF.

See: [Functional Benefits of UDFs](/docs/udf/udf_guide.html#functional-benefits-of-udfs).

## Queues using Lists

The List API provides atomic operations on unordered and ordered lists. Each client
implements the following server functions.

`Unordered [ 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233 ]`

### Queue
`append_items([377, 610], UNORDERED)`

### Dequeue
`remove_by_index_range(VALUE, 0, 1)`

For example, see the Python client's
[`aerospike_helpers.operations.list_operations`](https://aerospike-python-client.readthedocs.io/en/latest/aerospike_helpers.operations.html#module-aerospike_helpers.operations.list_operations) for the `list_append_items`, `list_get_by_index_range` and `list_remove_by_index_range` methods.
