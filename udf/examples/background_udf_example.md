---
title: Background UDF Example
description: An example running a background UDF to reset the TTL of records in a set.
---

In this example a user sees that after a code push new records are getting
accidentally set to never expire (TTL -1 from the client). The user wants to
identify these records and reset their TTL to 7 days.

We'll use a [predicate filter](/docs/guide/predicate.html)
to identify the records, and a background UDF to modify the TTL of matched records.

### Background UDF

A background UDF is a record UDF that modifies a record and returns no result.
As such, it can be attached to a scan or query to operate on record after
record.

**ttl.lua**

```lua
function set_ttl(rec, to_ttl)
  record.set_ttl(rec, to_ttl)
  aerospike:update(rec)
end
```

### Register the Module

In AQL:

```bash
aql> register module './ttl.lua'
OK, 1 module added.
```

### Predicate Filter

We'll use a predicate filter to find records with a void time of 0.

In the Java client:

```java
Statement stmt = new Statement();
stmt.setNamespace(params.namespace);
stmt.setSetName(params.set);
stmt.setPredExp(
  PredExp.recVoidTime(),
  PredExp.integerValue(0),
  PredExp.integerEqual()
  );

ExecuteTask task = client.execute(params.writePolicy, stmt, "ttl", "set_ttl", Value.IntegerValue(604800));
task.waitTillComplete();
```

### Without Using the Predicate Filter

If the language client doesn't have support for predicate filtering the logic to
identify the record can be moved into the UDF. This is less efficient, but
functionally the same.

**ttl.lua**

```lua
function modify_zero_ttl(rec, to_ttl)
  local rec_ttl = record.ttl(rec)
  if rec_ttl == 0 then
    record.set_ttl(rec, to_ttl)
    aerospike:update(rec)
  end
end
```

Now from AQL:

```bash
aql> register module './ttl.lua'
OK, 1 module added.

aql> execute ttl.modify_zero_ttl(604800) on test.foo
```
