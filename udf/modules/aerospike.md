

 Lua: aerospike Module

    Usage
    Methods
        aerospike:create()
        aerospike:update()
        aerospike:exists()
        aerospike:remove()

Usage

The aerospike object exposes a number of methods on the database. Among them is the ability to create or update a record.

If your function can be called for both non-existing and existing records, then it is best to perform a check on the record before calling create or update:
if aerospike:exists(rec) then
  aerospike:update(rec)
else
  aerospike:create(rec)
end
Methods

The following are methods of aerospike.
aerospike:create()

Create a new record in the database. If create() is successful, then return 0 (zero) otherwise it is an error.
Integer aerospike:create(Record r)
aerospike:update()

Update an existing record in the database. If update() is successful, then return 0 (zero) otherwise it is an error.
Integer aerospike:update(Record r)
aerospike:exists()

Checks for the existance of a record in the database. If the record exists, then true is returned, otherwise false.
Boolean aerospike:exists(Record r)
aerospike:remove()

Remove an existing record from the database. If delete() is success, then return 0 (zero), otherwise it is an error.
Integer aerospike:remove(Record r)

