---
title:  Annotated Record UDF Example
description:  Annotated Record UDF Example
---

The following example highlights some of the features, with explanations following the example code.

```lua
 1: function example_function(rec, userid, profile)
 2:   local ret = map()                     -- Initialize the return value (a map)
 3:
 4:   if not aerospike:exists(rec) then     -- Check to see that the record exists
 5:     ret['status'] = 'DOES NOT EXIST'    -- Set the return status
 6:   else
 7:     local x = rec['bin1']               -- Get the value from record bin named "bin1"
 8:                                         -- Use half the value from bin1 to
 9:     rec['bin2'] = (x / 2)               -- set a record bin named "bin2"
10:
11:    if  x < 0  then
12:      aerospike:remove(rec)              -- Delete the entire record
13:      ret['status'] = 'DELETE'           -- Populate the return status
14:    elseif  x > 100  then
15:      rec['bin3'] = nil                  -- Delete record bin named "bin3"
16:      ret['status'] = 'VALUE TOO HIGH'   -- Populate the return status
17:    else
18:      local myuserid = userid            -- Get the UDF argument "userid"
19:      local myprofile = profile          -- Get the UDF argument "profile"
20:      ret['status'] = 'OK'               -- Populate the return status
21:      ret['userdata'] = map{             -- Populate return value map
22:        myuserid    = userid,
23:        otheruserid = rec['userid'],
24:        match_score = rec['score']
25:      }
26:    end
27:
28:    aerospike:update(rec)                -- Update the main record
29:  end
30:
31:  return ret                             -- Return the Return value and/or status
32:end
```

### Explanation

Line 1 – The function declaration. 

The first argument is the record on which the function is to be applied. The remaining arguments are specific to the UDF, and are provided when the UDF is called.

Line 2 – Return Value

In this example, we return a Map from the UDF. This Map will be populated with a `status` field.

Line 4 – Test the existence of a record

In Aerospike, a Record UDF can be applied to an existing record or a non-existent record. This works because the key for a given record dictates where the record is stored in the cluster. This hashing of the key allows us to call the UDF on the node which is ultimately responsible for storing the record. With a Record UDF, you can choose to create a record if it doesn't exist.

Line 7 – Get the value from a bin in a record

The record object is accessed, and the data brought into a local variable. The different bins are only converted from internal C representation to Lua when accessed. The highest performance is achieved by not using the `record` object more than once, but using temporary variables.  You can get a value of a bin from a record using either of two different styles of access.  You can use index-based access:
```lua
local x = rec['bin1']
```
Or you can also use object-based access:
```lua
local x = rec.bin1
```
Line 9 – Set the value of a bin in a record

Setting ```record["bin2"] = r-value``` will result in the record's bin `bin2` being set.  You set the value of a bin from a record using index based access:
```lua
rec['bin2'] = (x / 2)
```
Or you can use object based access:
```lua
rec.bin2 = (x / 2)
```
Line 12 – Remove a Record

The aerospike object provides the remove() method for removing records from the database.

Line 15 – Remove a bin

You can remove a bin from a record by setting the value of the bin to nil.

Lines 21-25 – Creating a Map

This creates a new map and initializes it to the (key, value) pairs specified.  Any value can be returned to the client, including simple strings and integers. This example shows using a more complex return type: a [map](api/map.html). Lua tables - a combination of list and map type - *are not supported in the client*, so you must explicitly create a map() object, which will be converted to an Aerospike as_map() object on the client. You can use any name for this table.

Line 28 – Update a Record

After modifying the record, to persist the changes, you must call the `aerospike.update()` function to write to the database. If there is an error in writing, this function will return an error code. If you do not call this function, the record's changes will not be permanent. 

There is also a `create()` method, which will create a new record in the database if the record did not already exist.

Line 31 – Return

We return the map, which gets populated through out the function.  A simple mechanism (trick) to return a non-integer status from a UDF is to simply return it as a `status` bin. This returns an entire string to the client, which can be useful for easily returning different status in a programmer-friendly fashion.

