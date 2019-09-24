---
title: Deleting Records
description: Use the Aerospike PHP client to delete record and bin data from the Aerospike database.
---

Use these Aerospike PHP client methods to delete record and bin data from the Aerospike database:

- [remove()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_remove.md) &mdash; Remove the record at the specified key.
- [removeBin()](https://github.com/aerospike/aerospike-client-php/blob/master/doc/aerospike_removebin.md) &mdash; Remove specified bins from a record.
  - Remove one bin from a record:
  ```php
$key = $db->initKey("infosphere", "characters", 3);
$db->removeBin($key, ["Alma Mater"]);
  ```
  - Remove a record completely:
  ```php
$key = $db->initKey("infosphere", "characters", 5);
$status = $db->remove($key);
if ($status == Aerospike::OK) {
    echo "Record removed.\n";
} elseif ($status == Aerospike::ERR_RECORD_NOT_FOUND) {
    echo "No such record in the database.\n";
} else {
    echo "Error [{$db->errorno()}]: ".$db->error();
}
  ```

