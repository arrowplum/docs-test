---
title: Developing Record UDFs
description: Learn how to apply Record User-Defined Functions to a single record to augment read and write behavior within the Aerospike database.
assets: /docs/udf/assets
---

### Introduction

A Record UDF is a user-defined function that is applied to a **single record**.
Record UDFs can be used to augment both read and write behavior.

### Typical uses for Record-Based UDFs

* Record UDFs can be used to implement an atomic operation that does not
currently exist in the client. As a UDF will not perform at the same speed
and scale as native operations, it is important to consider whether the rich
[List](/docs/guide/cdt-list.html) and [Map](/docs/guide/cdt-map.html) API
methods cannot achieve the same effect. Combining multiple
native operations through the `operate()` command should also be considered,
before using a UDF.

* Record UDFs can be used for single-record, multi-operation transactions where
  data from one bin is used to determine the value of another bin.

* Record UDFs can be used to extend the existing list of
[predicate filtering](/docs/guide/predicate.html) operators for scan and
[query](/docs/guide/query.html).

* Background UDFs are good for large scale maintenance and data normalization,
  with a record UDF applied to all the records returned by a scan or query.

### General Comments on Record UDFs:

* The first argument of the function refers to the database record (e.g. `rec`).
Do not name the variable `record` as this is one of the types in the [Aerospike Lua API](/docs/udf/api_reference.html).
* Each subsequent argument is specific to the UDF, and must be one of the
types supported by the database: numeric (integer or double), string, list or map.
* A Record UDF should have one or more parameters (the record and optionally
  more). If a UDF has N parameters, and only (N-k) arguments passed in, the
last k will automatically be assigned a value of `nil`.
* A Record UDF may return one of the types supported by
the database: numeric (integer or double), string, bytes, list or map.
* A background UDF, where a single user-defined function is applied by scan or
query, modifies records without returning a result (write only).

### Record TTL and UDF

When creating or updating a record, its new TTL value adheres to the following hierarchy:
* If the UDF explicitly calls [`record.set_ttl()`](/docs/udf/api/record.html), that value
is used.
See [Setting time-to-live](https://discuss.aerospike.com/t/setting-time-to-live-ttl/4408).
* If there is a ttl specified in the policy field of the client UDF call, that
value is used. See [Expiration (Time to Live)](/docs/guide/policies.html#expiration-time-to-live-).
* If the client policy passes a TTL of 0, the namespace's
[default-ttl](/docs/reference/configuration/index.html#default-ttl) will be used.

### Example: Record Create or Update

In this simple example,
[ Annotated Record UDF Example ](/docs/udf/examples/record_udf_annotated.html),
we show how the UDF can create or update an Aerospike record.

### Example: String Slice Operator

Aerospike has two atomic operations for the [String](/docs/guide/data-types.html#string)
data type - append and prepend. The following record UDF adds an atomic string
slice operator.

```lua
function slice(rec, bin, a, b)
  local s = rec[bin]
  if type(s) == 'string' then
    return s:sub(a, b)
  end
  return nil
end
```

In aql

```bash
aql> register module 'util.lua'
OK, 1 module added.

aql> insert into test.foo (PK, x) values ('1', "Alright I like the beat except the snare, kick and keys")
OK, 1 record affected.

aql> execute util.slice('x', 9, 23) on test.foo where PK='1'
+-------------------+
| slice             |
+-------------------+
| "I like the beat" |
+-------------------+
1 row in set (0.001 secs)
```

### Example: Using Protobuf

In this example, a UDF makes use of an external Lua module, which in turns calls
a C shared object. The [Protobuf Module Example](/docs/udf/examples/proto_rb.html)
gives more details about the code involved in this example.

```bash
aql> select * from test.foo where PK='1'
Error: (2) AEROSPIKE_ERR_RECORD_NOT_FOUND

aql> execute pbuf.vartix(123, 'Wil', 'wil.jamieson@gmail.com') on test.foo where PK='1'
+--------+
| vartix |
+--------+
|        |
+--------+
1 row in set (0.001 secs)

aql> select * from test.foo where PK='1'
+----------------------------------------------------------------------------------------------+
| person                                                                                       |
+----------------------------------------------------------------------------------------------+
| 12 03 57 69 6C 08 7B 1A 16 77 69 6C 2E 6A 61 6D 69 65 73 6F 6E 40 67 6D 61 69 6C 2E 63 6F 6D |
+----------------------------------------------------------------------------------------------+
1 row in set (0.000 secs)

aql> execute pbuf.velkor() on test.foo where PK='1'
+--------+
| velkor |
+--------+
| 123    |
+--------+
1 row in set (0.000 secs)
```

### Example: Using a Background UDF to Reset TTLs

The [Background UDF Example](/docs/udf/examples/background_udf_example.html) resets the TTL of records
accidentally set to never expire.

