---
title: Transforming Records
description: Use the Aerospike C client to scan the records in the database and transform each record using UDFs. 
---

Use the Aerospike C client to scan the records in the database and transform each record using UDFs.

In SQL, a transformation on all rows in a table is performed as an `UPDATE` statement without predicates:

```cpp
UPDATE test.demo
SET a = 1, b = 2, c = 3
```

The Aerospike C client provides similar capabilities, but goes beyond what SQL provides by allowing you to transform records by applying a **Record UDF** on each record. These Record UDFs are applied to individual records, can have arguments, and can read and write the bins of a record and perform calculations.

Scanning and applying a Record UDF on each record can only be performed in the background; that is, the client sends the scan request to the database and does not wait for results. The scan is queued and runs on the database, and no results return to the client. The client has a scan ID, so can check status.

### Defining the Scan 

For scans that transform each record, define a UDF to execute on each record and execute the scan using the background scan operation, `aerospike_scan_background()`.

You build the scan just like in [Scan Records](/docs/client/c/usage/scan/scan.html), but use `as_scan_apply_each()`:

```cpp
as_scan scan;
as_scan_init(&scan, "test", "demo");
 
as_scan_apply_each(&scan, "mymodule", "mytransform", NULL);
```

Operation sequence:

1. Create an `as_scan` on the stack (line 1).
1. Initialize the scan to scan the Namespace _test_ within set _demo_ (line 2).
1. Add a Record UDF to apply to each record scanned. 

{{#note}}
The UDF `mytransform()` is defined in the `mymodule` UDF module. The UDF in this example is provided arguments.
{{/note}}

### Defining the Record UDF

The example uses the `mytransform` Record UDF defined in `mymodule` and mimics the SQL statement above.

```lua
function mytransform(rec)
    rec['a'] = 1
    rec['b'] = 2
    rec['c'] = 3
    aerospike:update(rec)
end
```

This more elaborate example increments _a_, calculates _b_ based on _a_, and calculates _c_ based on the values of _a_ and _b_:

```lua
function mytransform(rec)
    rec['a'] = rec['a'] + 1
    rec['b'] = rec['a'] * 2
    rec['c'] = rec['a'] + rec['b']
    aerospike:update(rec)
end
```

### Executing the Scan

To execute the scan using `aerospike_scan_background()`:

```cpp
uint64_t scan_id = 0;
if (aerospike_scan_background(&as, &err, NULL, &scan, &scan_id) != AEROSPIKE_OK) {
    fprintf(stderr, "error(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

`scan_id` queries the scan status. If `scan_id` = 0 (zero), the scan operation generates and sets the value of `scan_id`. 

### Checking Scan Status

To query scan status using `aerospike_scan_info()`: 

```cpp
as_scan_info scan_info;
if (aerospike_scan_info(&as, &err, NULL, scan_id, &scan_info) != AEROSPIKE_OK) {
    fprintf(stderr, "error(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

This populates `as_scan_info` with the scan status.

