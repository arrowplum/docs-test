---
title: Scan Records
description: Use the Aerospike C client APIs to scan all records in a specified namespace and set.
---

Use the Aerospike C client scan APIs to initialize and populate an `as_scan` object, and then use the following calls to execute a scan:

- `aerospike_scan_foreach()` &mdash; Execute the scan and call a function for each record.
- `aerospike_scan_background()` &mdash; Execute the scan without results, which provides the ability to check the scan status.

### Processing Results with `aerospike_scan_foreach()`

The `aerospike_scan_foreach()` call executes a scan, and calls a callback function for each record:

```cpp
typedef bool (*aerospike_scan_foreach_callback)(const as_val *value, void *udata);
```

`aerospike_scan_foreach()` is called with each record found during the scan, with _value_, and with the user-provided _udata_. Note that _value_ is `const as_val *`, which means that the callback is not responsible for destroying _value_, and _value_ is only available within the scope of the callback. The callback should not send _value_ outside of the scope of the callback.

The callback is called with a NULL _value_ when there are no more results.

This example is a normal scan operation, where _value_ is expected to be a record, and so casts _value_ into a record using `as_record_fromval()`:
- If _value_ is a record, then the function returns `as_record`. 
- If _value_ is not a record, then NULL returns. 

You can also use `as_val_type()` to check the _value_ data type. 

```cpp
bool callback(const as_val *value, void *udata) {
    if (value == NULL) {
        // scan is complete
        return true;
    }
  
    as_record *rec = as_record_fromval(value);
  
    if (rec != NULL) {
        // process record
    }
 
    return true;
}
```

### Checking Background Scan Status

Use `aerospike_scan_background()` to send a scan to the database to execute without the client waiting on results. The client is given a scan ID, used to check the status of the scan running in the database.

Use the scan ID to periodically check the scan status. Ideally, the application uses this information to determine how frequent to poll for status. 

The information about a scan is populated into an instance of `as_scan_info`:

```cpp
as_scan_info scan_info;
if (aerospike_scan_info(&as, &err, NULL, scan_id, &scan_info) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
```

This simple status check assumes that the check only occurs once. 

Applications that track elapsed time between scans can estimate how long a scan will take after polling for status by inspecting the `progress_pct` field of `as_scan_info`.

 
