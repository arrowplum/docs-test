---
title: Multiple Ops
description: Use the Aerospike C client to perform multiple operations on a record within a single transaction.
---

Applications can perform separate modification operations on multiple bins in a record within a single transaction using the Aerospike C client. This also allows modification of the bins of a record, which is read back to the client within the transaction (that is, it allows an application to perform an atomic modification that returns the result).

The following operations can be performed on a record:

Operation | Description | Conditions
--- | --- | ---
write | Write a value to a bin.  |
read  | Read the value of a bin.  | 
increment | Increment the integer value of a bin. |  Only integer values.
append  | Append a value to the value of a bin. | The value must be the same data type as the value in the bin; only bytes or strings are valid types.
prepend | Prepend a value to the value of a bin.  | The value must be the same data type as the value in the bin; only bytes or strings are valid types.
touch | Update the generation value of the record. | | 
list | Operations on a list bin.  See [List Operations](/apidocs/c/df/d6c/group__list__operations.html).
map | Operations on a map bin.  See [Map Operations](/apidocs/c/de/d4c/group__map__operations.html).

{{#note}}
Operations are performed in user defined order.
{{/note}}

### Tracking Page Views

The following application tracks the page views of a website. The key is the URL for a page. The record contains the following bins: 

- _last-updated_  (integer) &mdash; The time this record last updated.
- _views_  (integer) &mdash; The number of page view entries.
- _addr_ (byte array) &mdash; A sequence of IP address strings, delimited by NULL.
- _user_ (byte array) &mdash; A sequence of user ID strings, delimited by NULL.
- _time_ (byte array) &mdash; A sequence of timestamp strings, delimited by NULL.

The _addr_, _user_, and _time_ bins are sequences of NULL-separated strings. The entries in _addr_, _user_, and _time_ are in sync, such that a single page view is the value at the same index of each bin.

```cpp
as_operations ops;
as_operations_inita(&ops, 5);
as_operations_add_write_int64(&ops, "last-updated", timestamp);
as_operations_add_incr(&ops, "views", 1);
as_operations_add_append_raw(&ops, "addr", (uint8_t*)addr, strlen(addr) + 1);
as_operations_add_append_raw(&ops, "user", (uint8_t*)user, strlen(user) + 1);
as_operations_add_append_raw(&ops, "time", (uint8_t*)time, strlen(time) + 1);
 
as_key key;
as_key_init(&key, "app", "pages", url);
 
if (aerospike_key_operate(&as, &err, NULL, &key, &ops, &rec) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
 
as_operations_destroy(&ops);
```

### Increment-and-Read Operations

A common sequence of operations is increment-and-read. This allows an application to use a counter and read the values after each increment.

The following example is a page-view counter. The key of the record is the URL. The bin containing the counter is _views_.

{{#note}}
When performing a read, provide a record to populate with the bin being read.
{{/note}}

```cpp
as_operations ops;
as_operations_inita(&ops, 2);
as_operations_add_incr(&ops, "views", 1);
as_operations_add_read(&ops, "views");
 
as_record _rec;
as_record *rec = as_record_inita(&_rec, 1);
 
as_key key;
as_key_init(&key, "app", "pages", url);
 
if (aerospike_key_operate(&as, &err, NULL, &key, &ops, &rec) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
else {
    printf("views = %ld\n", as_record_get_int64(rec, "views", 0));
}
 
as_record_destroy(rec);
as_operations_destroy(&ops);
```

### Touching a Record

Each record contains metadata such as time-to-live and generation. The generation can be considered the version number of the record that increments with each update. The TTL is the time until the record expires. When reading a record, these values are not modified. So, if a record has a TTL of 5 minutes, even if it is constantly read, after 5 minutes it is no longer available. To keep the record from expiring, use the `touch` operation on the record.

The following example reads three bins and touches a record to prevent it from expiring. To read the three bins from the database, initialize a record with capacity for three bins.

```cpp
as_operations ops;
as_operations_inita(&ops, 4);
as_operations_add_touch(&ops);
as_operations_add_read(&ops, "x");
as_operations_add_read(&ops, "y");
as_operations_add_read(&ops, "z");
 
as_record _rec;
as_record *rec = as_record_inita(&_rec, 3);
 
as_key key;
as_key_init(&key, "app", "pages", url);
 
if (aerospike_key_operate(&as, &err, NULL, &key, &ops, &rec) != AEROSPIKE_OK) {
    fprintf(stderr, "err(%d) %s at [%s:%d]\n", err.code, err.message, err.file, err.line);
}
else {
    printf("x = %ld\n", as_record_get_int64(rec, "x", 0));
    printf("y = %ld\n", as_record_get_int64(rec, "y", 0));
    printf("z = %ld\n", as_record_get_int64(rec, "z", 0));
}
 
as_record_destroy(rec);
as_operations_destroy(&ops);
```

See _examples/basic\_examples/touch_ in the Aerospike C client package for examples of using `aerospike_key_operate()`.

### Multiple Results for the Same Bin

When multiple operations are performed on the same bin in the same record, the results are returned in a results array.  The array order is the same as the user-defined operation order.

This map example populates a hashmap bin and then performs various rank operations on that map.

```cpp
// Write map.
as_key rkey;
as_key_init_int64(&rkey, "test", "set", 4);

as_operations ops;
as_operations_inita(&ops, 1);

as_map_policy mode;
as_map_policy_init(&mode);

as_hashmap item_map;
as_hashmap_init(&item_map, 4);
as_string  mkey1;
as_integer mval1;
as_string_init(&mkey1, "Charlie", false);
as_integer_init(&mval1, 55);
as_hashmap_set(&item_map, (as_val*)&mkey1, (as_val*)&mval1);
as_string  mkey2;
as_integer mval2;
as_string_init(&mkey2, "Jim", false);
as_integer_init(&mval2, 98);
as_hashmap_set(&item_map, (as_val*)&mkey2, (as_val*)&mval2);
as_string  mkey3;
as_integer mval3;
as_string_init(&mkey3, "John", false);
as_integer_init(&mval3, 76);
as_hashmap_set(&item_map, (as_val*)&mkey3, (as_val*)&mval3);
as_string  mkey4;
as_integer mval4;
as_string_init(&mkey4, "Harry", false);
as_integer_init(&mval4, 82);
as_hashmap_set(&item_map, (as_val*)&mkey4, (as_val*)&mval4);

as_operations_add_map_put_items(&ops, "mapbin", &mode, (as_map*)&item_map);

as_error err;
as_record* rec = NULL;
as_status status = aerospike_key_operate(&as, &err, NULL, &rkey, &ops, &rec);
as_operations_destroy(&ops);

if (status != AEROSPIKE_OK) {
	return status;
}
as_record_destroy(rec);


// Perform rank operations.
as_operations_inita(&ops, 2);

// Get lowest score.
as_operations_add_map_get_by_rank(&ops, "mapbin", 0, AS_MAP_RETURN_VALUE);

// Get name/score of person with highest score.
as_operations_add_map_get_by_rank(&ops, "mapbin", -1, AS_MAP_RETURN_KEY_VALUE);

rec = NULL;
status = aerospike_key_operate(&as, &err, NULL, &rkey, &ops, &rec);
as_operations_destroy(&ops);

if (status != AEROSPIKE_OK) {
	return status;
}

as_bin* results = rec->bins.entries;

// First operation result contains lowest score.
int64_t lowest_score = results[0].valuep->integer.value;

// Second operation result contains name/score of person with highest score.
// The list contains two items, name and score.
as_list* list = &results[1].valuep->list;
const char* name = as_list_get_str(list, 0);
int64_t score = as_list_get_int64(list, 1);

printf("Highest score: %s/%d  Lowest score: %d\n", name, (int)score, (int)lowest_score);

as_record_destroy(rec);
```
