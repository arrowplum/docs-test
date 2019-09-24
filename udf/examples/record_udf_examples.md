---
title: Record UDF Examples
description:  Record UDF Examples
---

Several Record UDF examples have been created to demonstrate various
characteristics of Aerospike User-Defined Functions.

See the [general comments on Record UDFs](/docs/udf/developing_record_udfs.html#general-comments-on-record-udfs-).

### The Record UDF Examples

1. [Simple Annotated Record UDF](/docs/udf/examples/record_udf_annotated.html) - This annotated UDF
shows the common components of an average UDF.  Although the "function" of this UDF
is not particularly exciting or complex, it should serve as a good example.
It should be be relatively easy to take this starting example and modify it to
suit your needs for a beginning UDF.

2. [Statistics Example](/docs/udf/examples/record_udf_statistics.html) - Perform some statistical operations
on the record and then keep the accumulated results in the record.  Validate the
operations so that everything remains consistent.

3. [Background UDF Example](/docs/udf/examples/background_udf_example.html) - Uses a background UDF to identify records that fit a
certain criteria, and reset their TTL.

4. [Protobuf Module Example](/docs/udf/examples/proto_rb.html) - makes use of an external Protobuf Lua module, which in turns calls a Protobuf C shared object.
