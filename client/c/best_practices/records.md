---
title: Using Records
description: Work with records and the Aerospike C client on the Aerospike database.
---

In Aerospike, a record represents the data stored in the database. A record is composed of metadata and bins. Bins hold the record data. Each bin has a name and a value. Metadata is additional information about the record. Records are found in the database using keys.

### Initializing a Record

Use the following functions to initialize a record:

- `as_record_inita()` &mdash; Initialize a record and its bins on the stack.
- `as_record_init()` &mdash; Initialize a stack-allocated record, but allocate the bins on the heap.
- `as_record_new()` &mdash; Allocate and initialize a record and its bins on the heap.

Each method accepts an argument to specify the number of bins to allocate.

{{#note}}
Always use `as_record_destroy()` to release the record and related resources when they are no longer required. 
{{/note}}

#### Stack Allocated Record

Both `as_record_inita()` and `as_record_init()` initialize a stack-allocated 
record, but they allocate the bins of the record differently:

- `as_record_inita()` will allocate the bins on the stack.
- `as_record_init()` will allocate the bins on the heap.

The following is an example of `as_record_inita()`. Note that there is no difference in use, just in how allocation works.

```cpp
as_record rec;
as_record_inita(&rec, 3);
as_record_set_int64(&rec, 1);
as_record_set_int64(&rec, 2);
as_record_set_int64(&rec, 3);
 
as_record_destroy(&rec);
```

Even when the record and bins are on the stack, bin values can be allocated on the heap. Always use `as_record_destroy()` to release the record and related resources.

#### Heap Allocated Record

To allocate records on the heap using `as_record_new()`: 

```cpp
as_record *rec = as_record_new(10);
as_record_set_int64(rec, 1);
as_record_set_int64(rec, 2);
as_record_set_int64(rec, 3);
  
as_record_destroy(rec);
```

{{#note}}
Always use `as_record_destroy()` to release the record and related resources when they are no longer required. 
{{/note}}

### Accessing Bins of a Record

There are various methods for accessing the bins of a record. Start with an initialized record with bins. 

To put data into records:

- Initialize and populate the bins of a record. See Populating Bins of a Record.
- Read the record data returned by read operations. See Records and Read Operations.

Once the record is initialized, you can use any read functios to read the record.

#### Getting Bins

Use `get` functions to retrieve specific bin value types from a record. To access bins, each `get` function requires a record and the  bin name.

These `get` functions return the native value types stored in the bin:

- `as_record_get_int64()`
- `as_record_get_str()`

More details are provided in the API documentation. 

This example uses `getter` functions:

```cpp
int64_t ibin = as_record_get_int64(rec, "ibin", 123);
char *sbin   = as_record_get_str(rec, "sbin");
```

The following `get` functions return a pointer to a non-native value types:

- `as_record_get_integer()`
- `as_record_get_string()`
- `as_record_get_bytes()`
- `as_record_get_list()`
- `as_record_get_map()`

If the application saves a value pointer from these `get` functions and then destroys the record, the pointer then points to invalid data. To prevent this, increment the 'refcount' with the value returned in `as_val_reserve()`.  Some types (as_bytes, as_string) also need to have their internal values saved if used after calling as_record_destroy().  Examples:

```cpp
as_list* list = (as_list*)as_val_reserve(as_record_get_list(&rec, "lbin"));

as_map* map = (as_map*)as_val_reserve(as_record_get_map(&rec, "mbin"));

as_bytes* bbin = (as_bytes*)as_val_reserve(as_record_get_bytes(&rec, "bbin"));
uint8_t* bytes = bbin->value;
uint32_t len = bbin->size;

as_string* sbin = (as_string*)as_val_reserve(as_record_get_string(&rec, "sbin"));
char* str = sbin->value;

int64_t i = as_record_get_int64(&rec, "ibin")

as_record_destroy(&rec);

// process list, map, bytes, string and integer values and free resources afterwards.

as_list_destroy(list);
as_map_destroy(map);
free(bytes);
free(str);
```

#### Setting Bins

A record has a `set` function for each allowed data value type. Each `set` function requires the record, bin name, and bin value. Each `set` function returns `true` on success.

The following `set` functions are native value types:

- `as_record_set_int64()` &mdash; Set an int64_t value.
- `as_record_set_str()` &mdash; Set a NULL-terminated string value.
- `as_record_set_strp()` &mdash; Set a NULL-terminated string value to specify the value for when the record is destroyed.
- `as_record_set_raw()` &mdash; Set a byte array value.
- `as_record_set_rawp()` &mdash; Set a byte array value to free the value when the record is destroyed.

For details, see the API documentation. 

The following example demonstrates the `set` functions:

```cpp
as_record_set_int64(rec, "ibin", 123);
as_record_set_str(rec, "sbin", "abc");
as_record_set_strp(rec, "spbin", strdup("ijk"), true);
as_record_set_raw(rec, "rbin", (uint8_t*)"xyz", 3);
```

The following `set` functions point to non-native value types:

- `as_record_set_integer()` &mdash; Sets an `as_integer` value.
- `as_record_set_string()` &mdash; Sets an `as_string` value.
- `as_record_set_bytes()` &mdash; Sets an `as_bytes` value.
- `as_record_set_list()` &mdash; Sets an `as_list` value.
- `as_record_set_map()` &mdash; Sets an `as_map` value.

When a record is released using `as_record_destroy()`, the values set using these calls are released. If you are using these values elsewhere,  prevent their release by incrementing the `refcount` using `as_val_reserve()` when you set the value in the record.

The following example increments the `refcount` before adding it to the record. When the record is destroyed, the list remains intact. If `refcount` did not increment, the list would be destroyed so that the data no longer exists.

```cpp
as_arraylist list;
as_arraylist_init(&list, 3);
as_arraylist_append_int64(&list, 1);
as_arraylist_append_int64(&list, 2);
as_arraylist_append_int64(&list, 3);
 
as_record rec;
as_record_inita(&rec, 1);
as_record_set_list(&rec, "lbin", (as_list*)as_val_reserve(&list));
 
as_record_destroy(&rec);
 
int64_t i1 = as_arraylist_get_int64(&list, 1);
int64_t i2 = as_arraylist_get_int64(&list, 2);
int64_t i3 = as_arraylist_get_int64(&list, 3);
 
as_arraylist_destroy(&list);
```

Lines 13&ndash;15 result in valid reads of data. If `as_val_reserve()` is not called, the result is invalid reads, which leads to data corruption.

#### Setting the Generation

The generation value contains the record version. Each modification to a record increments the generation value. You can set the generation value of a record; however it is mostly used to tell the server the expected generation of the record.

```cpp
rec->gen = 0
```

#### Setting the Time-To-Live (TTL)

A record also has a time-to-live (TTL) value, which specifies record expiration or when to evict it from the database. The value is defined 
as the number of seconds from now.

```cpp
rec->ttl = 3600 * 24;
```
<a name=TraverseBin>
### Traversing Bins of a Record
</a>

To traverse the bins of a record, the Aerospike C client provides the following methods:

- `as_record_foreach()` &mdash; Iterate over each bin of a record, calling a function for each bin.
- `as_record_iterator()` &mdash; The Iterator API for record bins.

**`as_record_foreach`**

`as_record_foreach()` iterates over the bins of a record, and calls a function for each bin. This function also takes a user-data argument, which is any data the user provides for use within the callback function.

This example populates a record, then iterates over the bins:

```cpp
as_record rec;
as_record_inita(&rec, 3);
as_record_set_int64(&rec, "a", 1);
as_record_set_int64(&rec, "b", 2);
as_record_set_int64(&rec, "c", 3);
 
as_record_foreach(&rec, callback, NULL);
```

The callback for this example prints the name and value of the bin:

```cpp
bool callback(const char *name, const as_val *value, void *udata)
{
    as_integer *ivalue = as_integer_fromval(value);
  
    if (ivalue) {
        printf("%s = %d\n", name, as_integer_get(ivalue));
    }
    else {
        printf("%s is not an integer?!\n", name);
    }
  
    return true;
}
```

If the callback returns `true`, then iteration continues to the next bin, otherwise iteration stops.

**`as_record_iterator`**

The `as_record_iterator` data structure provides the ability to iterate over the bins of a record. 

These calls initialize `as_record_iterator` functions:

- `as_record_iterator_init()` &mdash; Initializes a stack-allocated iterator.
- `as_record_iterator_new()` &mdash; Allocates and initializes a heap-allocated iterator.

Once initialized, use the following function to traverse the iterator:

- `as_record_iterator_has_next()` &mdash; Tests if there are more bins available for iteration.
- `as_record_iterator_next()` &mdash; Progresses the iterator to the next bin, returning the bin.

The following example uses the same record:

```cpp
as_record_iterator it;
as_record_iterator_init(&it, &rec);
 
while (as_record_iterator_has_next(&it)) {
    as_bin *bin        = as_record_iterator_next(&it);
    char *name         = as_bin_get_name(bin);
    as_val *value      = (as_val*)as_bin_get_value(bin);
    as_integer *ivalue = as_integer_fromval(value);
  
    if (ivalue) {
        printf("%s = %d\n", name, as_integer_get(ivalue));
    }
    else {
        printf("%s is not an integer?!\n", name);
    }
}
```

### Records and Read Operations

Read operations populate records with the data received from the server.

These functions read records from the server:

- `aerospike_key_get()` &mdash; Reads all the bins of a record. See Reading Records.
- `aerospike_key_select()` &mdash; Reads the specified bins of a record. See Reading Records.
- `aerospike_key_operate()` &mdash; Performs operations on a record. These can include a read operation on specific bins. See Record Operations.

Each operation accepts a record argument (an `as_record`), which is then populated with the data from the server.

The following sections explain how a record is populated by read operations. 

#### NULL Records

If the record argument is a NULL pointer, then the read operation allocates a record on the heap, and initializes it with enough bins to hold the all the bins returned from the server.

```cpp
as_record *rec = NULL;
 
aerospike_key_get(&as, &err, NULL, &key, &rec)
 
as_record_destroy(rec);
```

{{#note}}
Always use `as_record_destroy()` to release the record and related resources when they are no longer required. 
{{/note}}

#### Initialized Records

You can use an initialized record as the argument to a read operation. If the record was initialized with bins (more than zero), then a read operation attempts fill as many available bins of the record as possible. 

For example, a record was initialized with 10 bins:
- If the server returns 3 bins, then only 3 of the record's bins can be populated. 
- If the server returned 20 bins, then the first 10 bins read are added to the record. 
- If the record only has 5 of its 10 bins available when passed to the read operation, only 5 bins can be filled from the server. 
- If the record was initialized without bins, the read operation attempts to allocate enough bins on the heap to store all bins retrieved from the server.

The following functions initialize a record either on the stack or on the heap:

- `as_record_inita()` &mdash; Initialize a record and its bins on the stack.
- `as_record_init()` &mdash; Initialize a stack-allocated record, but allocate the bins on the heap.
- `as_record_new()` &mdash; Allocate and initialize a record and its bins on the heap.

