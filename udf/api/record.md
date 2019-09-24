---
title: record
description: Aerospike's Record API is provided by the UDF system and can represent an existing record or an empty record, and gives access to a record's bins and metadata.
---

### Usage

{{#note}}
For use-cases that have [`single-bin`](/docs/reference/configuration#single-bin) configuration set to `true`, the bin name is required to be an empty string for reading or writing the bin for versions 3.15 and above. Versions prior to 3.15 do not support UDFs on single-bin namespaces.
{{/note}}

A Record is provided by the UDF system and can represent an existing record or an empty record. It provides access to a record's bins and metadata.

A Record can be accessed much like a Map or Lua Table, where the key can be a String:

```lua
> rec
[ REC ]
 
> rec['a'] = 1
> rec['a']
1
```

A Record must only contain values of the following types:

- integer
- string
- [bytes](/docs/udf/api/bytes.html)
- [list](/docs/udf/api/list.html)
- [map](/docs/udf/api/map.html)

Placing other Lua types - for example, functions or tables - will result in run-time errors.

Changes to an existing Record are not reflected in the database unless you call aerospike:update(rec) to write the Record back to the database.

### Functions

-------------------------------------------------------------------------------
#### record.bin_names()

Get the bin names of a record.

```lua
function record.bin_names(r: Record): Lua table
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the bin names of.
    </ul>
  </dd>
  <dt>Returns
  <dd>A Lua table containing the bin names.  This table can be iterated over with the ipairs() function.
</dl>

Example:

```lua
> names = record.bin_names(rec)
> for i, name in ipairs(names) do
> 	info("bin %d name = %s", i, tostring(name))
> end
bin 1 name = bin1
bin 2 name = bin2
bin 3 name = third bin
```


-------------------------------------------------------------------------------
#### record.digest()

Get the digest of a record. The digest is the hash of the key and other values using for distributing the record across the cluster. 

```lua
function record.digest(r: Record): Bytes
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the digest value of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The digest of the record.
</dl>

Example:

```lua
> record.digest(rec)
Bytes(68656c6c6f20776f726c64)
```


-------------------------------------------------------------------------------
#### record.gen()

Get the generation value of a record. The generation values is equivalent to a revision number.

```lua
function record.gen(r: Record): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the generation value of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The generation of the record.
</dl>

Example:

```lua
> record.gen(rec)
5
```


-------------------------------------------------------------------------------
#### record.key()

Get the key of a record.

```lua
function record.key(r: Record): string
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the key of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The key of the record.  Returns nil if the record's key is not stored.
</dl>

Example:

```lua
> record.key(rec)
my-key
```


-------------------------------------------------------------------------------
#### record.last_update_time()

Get the last update time of a record. Since Server version 3.8.3

```lua
function record.last_update_time(r: Record): integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the last update time of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The last update time of the record, expressed in milliseconds since the Citrusleaf epoch (00:00:00 UTC on 1 Jan 2010).
</dl>

Example:

```lua
> record.last_update_time(rec)
226179209266
```


-------------------------------------------------------------------------------
#### record.numbins()

Get the number of bins in a record.

```lua
function record.numbins(r: Record): integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the number of bins of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The number of bins in the record.  Returns 1 if the record belongs to a single-bin namespace.
</dl>

Example:

```lua
> record.numbins(rec)
7
```


-------------------------------------------------------------------------------
#### record.setname()

Get the set name of a record.

```lua
function record.setname(r: Record): string
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the set name of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The set name of the record.  Returns nil if the record does not belong to a set.
</dl>

Example:

```lua
> record.setname(rec)
my-set
```


-------------------------------------------------------------------------------
#### record.ttl()

Get the time-to-live (ttl) of a record

```lua
function record.ttl(r: Record): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to get the time-to-live (ttl) value of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The ttl of the record, in seconds.
</dl>

Example:

```lua
> record.ttl(rec)
40000
```


-------------------------------------------------------------------------------
#### record.set_ttl()

Set the time-to-live (ttl) of a record.  When the record is updated this ttl will take effect.

```lua
function record.set_ttl(r: Record, ttl: Integer): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to set the time-to-live value for.
      <li>`ttl` – the time-to-live value in seconds.
    </ul>
  </dd>
</dl>

Example:

```lua
> record.set_ttl(rec, 10)
```

*known issue* - reading the ttl within the same UDF transaction context, does not return the updated ttl, even though set_ttl() and update() are called. Subsequent transaction call will correctly return the ttl.

-------------------------------------------------------------------------------
#### record.drop_key()

Discard a record's key.  When the record is updated its key will no longer be stored.

```lua
function record.drop_key(r: Record): nil
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` whose key is to be dropped.
    </ul>
  </dd>
</dl>

Example:

```lua
> record.drop_key(rec)
```
