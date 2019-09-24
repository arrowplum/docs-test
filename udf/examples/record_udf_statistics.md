---
title: UDF Example – Statistics
description: UDF Example – Statistics
---

This Sample UDF Package `udf_samples.lua` defines the external function `update_stats()`.  Lua functions are either internal (e.g. local function `foo()` ) or external (e.g. function `update_stats()`).  If a function is local, then it can be called ONLY from inside the lua file.  It is not visible outside and cannot be called externally.

The `update_stats()` function is pretty simple:

If the main record does not already exist, then the calculations (max, min, sum, ave, count) are all trivial.  We set the values and create the record.  Note that when a UDF is called that references a record that does not yet exist, the UDF has the option of creating that record.  The UDF could also return with a `does not exist` value, if that is the preferred behavior.

Otherwise, if the record does exist, then we do the math calculations (max, min, sum, ave, count) and then update the record.

Note the use of Aerospike Logging : trace(" ENTER "), trace(" EXIT "), warn( "ERROR" ).

### Statistics Example
	
```lua
-- ======================================================================
-- UDF SAMPLES (udf_samples.lua)
-- ======================================================================
-- These samples are used for Aerospike Documentation.  They are included
-- here to validate that they are correct Lua -- and actually run.
 
 
-- ======================================================================
-- update_stats()
-- ======================================================================
-- This UDF manages a set of statistics for the user.  It tracks several
-- values (Min, Max, Ave, Sum, Count) for the user.
--
-- The statistics record has the following bins:
-- MaxValue:   holds the largest number seen so far.
-- MinValue:   holds the smallest number seen so far.
-- Sum:        holds the sum of what's been seen so far.
-- AveValue:   holds the average value (computed)
-- ValueCount: holds the number of values that have been processed
--
-- Parms:
-- (*) rec: the main Aerospike Record
-- (*) newValue: the new value passed in by the user
-- Return:
-- (Success): 0
-- (Error): The error code returned by the "error()" call
-- ======================================================================
function update_stats( rec, newValue )
  local meth = "update_stats()";
  trace("[ENTER]<%s> Value(%s) valType(%s)", meth, tostring(newValue),type(newValue));
  local rc = 0;
 
  -- Check for the record, create if not yet there.
  if( not aerospike:exists( rec ) ) then
    rc = aerospike:create( rec );
    if( rc == nil or rec == nil ) then
      warn("[ERROR]<%s> Problems creating record", meth );
      error("ERROR creating record");
    end
    info("<CHECK><%s> Check Create Result(%s)", meth, tostring(rc));
    -- It's a new record, so everything starts out fresh
    rec["MaxValue"]   = newValue;
    rec["MinValue"]   = newValue;
    rec["Sum"]        = newValue;
    rec["AveValue"]   = newValue;
    rec["ValueCount"] = 1;
  else
    -- It's an existing record, so do the math operations.  However, we still
    -- have to check to see if each bin exists (the record might exist but
    -- the bins could be nil).  We will test them individually, but we expect
    -- that if one bin is nil, they probably all are.
    local sum = 0;
    local count = 1;
    if( rec["MaxValue"] == nil or newValue > rec["MaxValue"] ) then
      rec["MaxValue"] = newValue; -- write the value if new or changed
    end
    if( rec["MinValue"] == nil or newValue < rec["MinValue"] ) then
      rec["MinValue"] = newValue; -- write the value if new or changed
    end
    if( rec["ValueCount"] == nil ) then
      count = 1;                  -- this is the first one
    else
      count = rec["ValueCount"];
      count = count + 1;          -- increment the value
    end
    if( rec["Sum"] == nil or type(rec["Sum"]) ~= "number" ) then
      sum = newValue;
    else
      sum = rec["Sum"];
      sum = sum + newValue;       -- Notice that we do NOT do this: rec["sum"] = rec["sum" + 1;
    end
    rec["ValueCount"] = count;
    rec["Sum"] = sum;
    rec["AveValue"] = sum / count;
  end -- else record already exists
   
  -- All done -- update the record (checking for errors on update)
  rc = aerospike:update( rec );
  if( rc ~= nil and rc ~= 0 ) then
    warn("[ERROR]<%s> record update: rc(%s)", meth,tostring(rc));
    error("ERROR updating the record");
  end
  rc = 0; -- safety, in case rc == nil.  But, no error here.
  trace("[EXIT]:<%s> RC(%d)", meth, rc );
  return rc;
end -- update_stats()
```

