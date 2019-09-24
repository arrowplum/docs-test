---
title: Scan Applying a Record UDF
description: Use the Aerospike PHP client to apply a Record UDF to all records in a set.
---

Use the following Aerospike PHP client APIs to apply a UDF to each record in a set in the background:

- [scanApply()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_scanapply.md)
- [scanInfo()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_scaninfo.md)

{{#note}}
UDFs must register with the Aerospike server.
{{/note}}

### Running the Background Scan

This example uses the registered a UDF module named *rec_udf* with the Lua function `accumulate`:

```lua
function accumulate(rec, collector, source)
    rec[collector] = rec[collector] + rec[source]
    aerospike:update(rec)
end
```

To apply this UDF to all the records of a specified set:

```php
$status = $db->scanApply("test", "users", "rec_udf", "accumulate", ["compensation", "income"], $scan_id);
if ($status == Aerospike::OK) {
    echo "The background scan ID is $scan_id\n";
    while(true) {
        $status = $db->scanInfo($scan_id, $info);
        if ($status == Aerospike::OK) {
            var_dump($info);
            if ($info["status"] === Aerospike::SCAN_STATUS_COMPLETED) {
                echo "Background scan is complete!";
                break;
            }
        }
        sleep(1);
    }
} else {
    echo "An error occured while launching the background scan [{$db->errorno()}] ".$db->error();
}
```

The records update and _compensation_ adds _income_ to its value.

