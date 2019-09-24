---
title: aerospike
description: Learn how to use the Aerospike object to expose a number of methods on the database, including the ability to create or update a record.
---

### Usage

The aerospike object exposes a number of methods on the database. Among them is the ability to create or update a record.

If your function can be called for both non-existing and existing records, then it is best to perform a check on the record before calling create or update:

```lua
if aerospike:exists(rec) then
  aerospike:update(rec)
else
  aerospike:create(rec)
end
```

### Methods

The following are methods of aerospike.


-------------------------------------------------------------------------------
#### aerospike:create()

Create a new record in the database. If create() is successful, then return 0 (zero) otherwise it is an error.

```lua
function aerospike:create(r: Record): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to create.
    </ul>
  </dd>
  <dt>Returns
  <dd>The status code of the operation.
</dl>


-------------------------------------------------------------------------------
#### aerospike:update()

Update an existing record in the database. If update() is successful, then return 0 (zero) otherwise it is an error.

```lua
function aerospike:update(r: Record): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to update.
    </ul>
  </dd>
  <dt>Returns
  <dd>The status code of the operation.
</dl>


-------------------------------------------------------------------------------
#### aerospike:exists()

Checks for the existance of a record in the database. If the record exists, then true is returned, otherwise false.

```lua
function aerospike:exists(r: Record): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to test the existence of.
    </ul>
  </dd>
  <dt>Returns
  <dd>`True` if the record exists, otherwise `False`.
</dl>


-------------------------------------------------------------------------------
#### aerospike:remove()

Remove an existing record from the database. If delete() is success, then return 0 (zero), otherwise it is an error.

```lua
function aerospike:remove(r: Record): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`r` – the `Record` to remove.
    </ul>
  </dd>
  <dt>Returns
  <dd>The status code of the operation.
</dl>

