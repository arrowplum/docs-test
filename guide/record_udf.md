---
title: Record UDF 
description: Record UDFs operate on single record, and can execute on result record sets created by other system features such as scan and secondary index queries.
assets: /docs/guide/assets
---
<div style="float: right" >
![]({{book.assets}}/UDF_record.png)
</div>

Record UDFs perform operations on a single record such as creating or updating records based on a set of parameters. Background UDFs are record UDFs applied to the records returned from scans and secondary index queries.

See [Typical uses for Record-Based UDFs](/docs/udf/developing_record_udfs.html#typical-uses-for-record-based-udfs).

## Implementation

To build and run a Record UDF on an Aerospike Server cluster:

1. [Install](/docs/operations/install/index.html) and configure the [Aerospike server](/download).
2. Write your UDF. UDFs are written in the [Lua programming language](http://www.lua.org/). See the [UDF Developer Guide](/docs/udf/udf_guide.html).
3. Register the UDF with the cluster.
4. Invoke the UDF in your application.
5. Execute the UDF on a record or a set of records ([secondary index query](/docs/guide/query.html#secondary-index) or database [scan](/docs/guide/scan.html))


### Writing Record UDFs

**ops.lua**
```lua
local function factorial_tail(i, acc)
  if i == 0 then
    return acc
  end
  return factorial_tail(i - 1, acc * i)
end

function factorial(rec, bin)
    if aerospike:exists(rec) then
        local v = rec[bin]
        if type(v) == 'number' then
            return factorial_tail(v, 1)
        end
    end
    return nil
end
```

{{#note}}
See [Developing Record UDFs](/docs/udf/developing_record_udfs.html).
{{/note}}

{{#warn}}
To ease diagnosing test errors during the development process, develop UDFs using a single server node.
{{/warn}}

### Registering Record UDFs

<div style="float: right" >
![]({{book.assets}}/UDF_Register.png)
</div>

UDFs must register with the Aerospike cluster before calling functions. UDFs only have to register with one node in the cluster. 
<br>
<br>
<br>
<br>
<br>
<br>

The interactive [`aql`](/docs/tools/aql/udf_management.html) CLI is a convenient way to register and execute UDFs.

```bash
aql> register module 'ops.lua'
OK, 1 module added.
```

Aerospike replicates registered UDFs through the entire cluster. Registered UDFs also persist on the node, so they persist through node and machine restarts. 

{{#note}}
See [Managing UDFs](/docs/udf/managing_udfs.html).
{{/note}}

## Executing Record UDFs
<div style="float: right" >
![]({{book.assets}}/UDF_Execute.png)
</div>

Language-specific examples are available in the following Aerospike client libraries: 

* [Java](/docs/client/java/usage/udf/apply.html)
* [C# .NET](/docs/client/csharp/usage/udf/apply.html)
* [Node.js](/docs/client/nodejs/usage/udf/apply.html)
* [PHP](/docs/client/php/usage/udf/apply.html)
* [Python](/docs/client/python/usage/udf/apply.html)
* [Go](/docs/client/go/usage/udf/apply.html)
* [Ruby](/docs/client/ruby/usage/udf/apply.html)
* [C](/docs/client/c/usage/udf/apply.html)

## Executing Record UDFs from aql

```bash
aql> INSERT INTO test.demo (PK, i) VALUES ('r1', 4)
OK, 1 record affected.

aql> EXECUTE ops.factorial('i') ON test.demo WHERE PK='r1'
+-----------+
| factorial |
+-----------+
| 24        |
+-----------+
1 row in set (0.000 secs)

aql> INSERT INTO test.demo (PK, i) VALUES ('r2', 9)
OK, 1 record affected.

aql> EXECUTE ops.factorial('i') ON test.demo WHERE PK='r2'
+-----------+
| factorial |
+-----------+
| 362880    |
+-----------+
1 row in set (0.001 secs)

aql> INSERT INTO test.demo (PK, i) VALUES ('r3', 0)
OK, 1 record affected.

aql> EXECUTE ops.factorial('i') ON test.demo WHERE PK='r3'
+-----------+
| factorial |
+-----------+
| 1         |
+-----------+
1 row in set (0.001 secs)

aql> INSERT INTO test.demo (PK, i) VALUES ('r4', 'llama')
OK, 1 record affected.

aql> EXECUTE ops.factorial('i') ON test.demo WHERE PK='r4'
+-----------+
| factorial |
+-----------+
|           |
+-----------+
1 row in set (0.000 secs)

```

## Examples

{{#note}}
See [Record UDF Examples](/docs/udf/examples/record_udf_examples.html).
{{/note}}

## Known Limitations

- The Aerospike Record UDF system supports only single return values.

  Although Lua allows multple values to return as function result values, the final UDF returning to the client application can only contain a single object as the return value. Use lists to return multiple objects.
- The Aerospike Record UDF system does not allow multi-record operations.

    For example, using UDFs that on writes to a particular key update other keys, or to request values from other keys to simulate joins. In Aerospike, keys can be on other servers, so these functions require asynchronous operations, which can cause deadlocks.
- Aerospike only supports limited sandboxing to protect the system from crashes.

    Aerospike provides time limits to terminate individual UDF invocations runnning longer than the set time. However, it is still possible (especially by calling a C module from Lua or very high memory usage) to crash the entire server.
- Aerospike does not support Record UDF on batch results.

## References

- [UDF Developer Guide](/docs/udf/udf_guide.html)
- [UDF Library API Reference Guide](/docs/udf/api_reference.html).
