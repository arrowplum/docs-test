---
title: aql – Record Operations
description: Learn how aql can be used to insert and delete records.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: aql
    url: /docs/v3/aql.html
---

### Insert a Record

The following is the command for inserting a record:

```sql
INSERT INTO <ns>[.<set>] (PK, <bins>) VALUES (<key>, <values>)
```

Where

- `<ns>` is the namespace for the record.
- `<set>` is the set name for the record.
- `<key>` is the record's primary key.
- `<bins>` is a comma-separated list of bin names.
- `<values>` is comma-separated list of bin values, which may include type cast expressions. Set to NULL (case insensitive & w/o quotes) to delete the bin.

Example:

```sql
aql> INSERT INTO test.testset (PK, a, b) VALUES ('xyz', 'abc', 123)
```

### Delete a Record

The following is the command for deleting a record:

```sql
DELETE FROM <ns>[.<set>] WHERE PK=<key>
```

Where

- `<ns>` is the namespace for the record.
- `<set>` is the set name for the record.
- `<key>` is the record's primary key.

Example:

```sql
aql> DELETE FROM test.testset WHERE PK='xyz'
```

### Operate on a Record

The following is the command for inserting a record:

```sql
OPERATE <op(<bin>, params...)>[with_policy(<map policy>),] [<op(<bin>, params...)> with_policy (<map policy>) ...] ON <ns>[.<set>] where PK=<key>
```

Where
- `<op>` name of operation to perform.
- `<bin>` is the name of a bin.
- `<params>` parameters for operation.
- `<map policy>` map operation policy.
- `<ns>` is the namespace for the records to be queried.
- `<set>` is the set name for the record to be queried.
- `<key>` is the record's primary key.

Example:

```sql
aql> OPERATE LIST_APPEND(listbin, 1), LIST_APPEND(listbin2, 10) ON test.demo where PK = 'key1'
aql> OPERATE LIST_POP_RANGE(listbin, 1, 10) ON test.demo where PK = 'key1'
```

Supported OPs

```bash
	LIST_APPEND (<bin>, <val>)         
	LIST_INSERT (<bin>, <index>, <val>)
	LIST_SET    (<bin>, <index>, <val>)
	LIST_GET    (<bin>, <index>)       
	LIST_POP    (<bin>, <index>)       
	LIST_REMOVE (<bin>, <index>)       
	LIST_APPEND_ITEMS (<bin>, <list of vals>)         
	LIST_INSERT_ITEMS (<bin>, <index>, <list of vals>)
	LIST_GET_RANGE    (<bin>, <startindex>[, <count>])
	LIST_POP_RANGE    (<bin>, <startindex>[, <count>])
	LIST_REMOVE_RANGE (<bin>, <startindex>[, <count>])
	LIST_TRIM         (<bin>, <startindex>[, <count>])
	LIST_INCREMENT    (<bin>, <index>, <numeric val>) 
	LIST_CLEAR        (<bin>) 
	LIST_SIZE         (<bin>) 
	MAP_PUT             (<bin>, <key>, <val>) [with_policy (<map policy>)]
	MAP_PUT_ITEMS       (<bin>, <map>)  [with_policy (<map policy>)]
	MAP_INCREMENT       (<bin>, <key>, <numeric val>) [with_policy (<map policy>)]
	MAP_DECREMENT       (<bin>, <key>, <numeric val>) [with_policy (<map policy>)]
	MAP_GET_BY_KEY      (<bin>, <key>)  
	MAP_REMOVE_BY_KEY   (<bin>, <key>)  
	MAP_GET_BY_VALUE    (<bin>, <value>)
	MAP_REMOVE_BY_VALUE (<bin>, <value>)
	MAP_GET_BY_INDEX    (<bin>, <index>)
	MAP_REMOVE_BY_INDEX (<bin>, <index>)
	MAP_GET_BY_RANK     (<bin>, <rank>) 
	MAP_REMOVE_BY_RANK  (<bin>, <rank>) 
	MAP_REMOVE_BY_KEY_LIST    (<bin>, <list of keys>)         
	MAP_REMOVE_BY_VALUE_LIST  (<bin>, <list of vals>)         
	MAP_GET_BY_KEY_RANGE      (<bin>, <startkey>, <endkey>)   
	MAP_REMOVEBY_RANGE        (<bin>, <startkey>, <endkey>)   
	MAP_GET_BY_VALUE_RANGE    (<bin>, <startval>, <endval>)   
	MAP_REMOVE_BY_VALUE_RANGE (<bin>, <startval>, <endval>)   
	MAP_GET_BY_INDEX_RANGE    (<bin>, <startindex>[, <count>])
	MAP_REMOVE_BY_INDEX_RANGE (<bin>, <startindex>[, <count>])
	MAP_GET_BY_RANK_RANGE     (<bin>, <startrank> [, <count>])
	MAP_REMOVE_BY_RANK_RANGE  (<bin>, <startrank> [, <count>])
	MAP_CLEAR     (<bin>) 
	MAP_SET_TYPE  (<bin>, <map type>) 
	MAP_SIZE      (<bin>) 
	TOUCH   ()            
	READ    (<bin>)       
	WRITE   (<bin>, <val>)
	PREPEND (<bin>, <val>)
	APPEND  (<bin>, <val>)
	INCR    (<bin>, <numeric val>)
```

Map Policy - storage order:

|Policy| Description|
|------|-----------------------------------------------|
|AS_MAP_UNORDERED|Map is not ordered. (default)|
|AS_MAP_KEY_ORDERED|Order map by key|
|AS_MAP_KEY_VALUE_ORDERED|Order map by key, then value.|


Map Policy - write mode:


|Policy| Description|
|------|-----------------------------------------------------------|
|AS_MAP_UPDATE|An item will be created or overwritten if key exists|
|AS_MAP_UPDATE_ONLY|If the key does not exist, the write will fail.|
|AS_MAP_CREATE_ONLY|If the key already exists, the write will fail.|


For details refer to: 
- [list Ops](/docs/guide/cdt-list.html)
- [map Ops](/docs/guide/cdt-map.html)

### Get a Record using DIGEST

For tools version 3.5.11 and above, the following command can be used or getting a record using the DIGEST:

When providing the HEX representation of the digest (for example from the server logs), use *DIGEST* :

```sql
SELECT * FROM <ns>[.<set>] WHERE DIGEST='<DIGEST_HEX_STRING>'
```

When providing the Base64 representation of the digest (for example from asbackup file), use *EDIGEST* :

```sql
SELECT * FROM <ns>[.<set>] WHERE EDIGEST='<DIGEST_B64_STRING>'
```

Where

- `<ns>` is the namespace for the record.
- `<set>` is the set name for the record.
- `<DIGEST_HEX_STRING>` is the Hexadecimal representation of the record's digest.
- `<DIGEST_B64_STRING>` is the Base64 representation of the record's digest.

Example:

```sql
aql> SELECT * FROM test.testset where DIGEST='139FE89822B63DFC173AEA51CCF2EF091AB3129F'
+---------+---------+-----------------------------------+---------------+-------+
| bin1    | bin2    | bin3                              | LDTCONTROLBIN | binl1 |
+---------+---------+-----------------------------------+---------------+-------+
| "val01" | "val02" | ["string1", "string2", "string3"] |               |       |
+---------+---------+-----------------------------------+---------------+-------+
1 row in set (0.000 secs)
```

```sql
aql> SELECT * FROM test.testset where EDIGEST='E5/omCK2PfwXOupRzPLvCRqzEp8='
+---------+---------+-----------------------------------+---------------+-------+
| bin1    | bin2    | bin3                              | LDTCONTROLBIN | binl1 |
+---------+---------+-----------------------------------+---------------+-------+
| "val01" | "val02" | ["string1", "string2", "string3"] |               |       |
+---------+---------+-----------------------------------+---------------+-------+
1 row in set (0.000 secs)
```


### Define Datatype

The datatype of a bin value (e.g. MAP, LIST, GeoJSON etc) needs to be specified when the record is created. Once created, all the record operations such as INSERT, DELETE and queries are valid. The two options of specifying are as follows:


1) Using the datatype by defining itself as is.

```sql
INSERT INTO test.testset(PK, a,b) VALUES ('xyz9', 'abc10', MAP('{"map":1, "of":2, "items":3}'))

INSERT INTO test.demo (PK, gj) VALUES ('key1', GEOJSON('{"type": "Point", "coordinates": [123.4, -456.7]}'))
```

1) Using the CAST expression

Use the CAST expression to define the datatype.

```sql
INSERT INTO test.testset (PK, a, b) VALUES ('xyz11', 'abc11', CAST('{"map":100, "of":200, "items":300}' AS MAP))

INSERT INTO test.testset(PK, a,b) VALUES ('xyz12', 'abc12', CAST('{"type": "Point", "coordinates": [123.4, -456.7]}' as GEOJSON))
```

### Truncate Data

The following is the command for deleting all the data in a namespace or a set:

```sql
TRUNCATE <ns>[.<set>] [upto <LUT>] 
```

Where

- `<ns>` is the namespace to truncate.
- `<set>` is the set name to truncate.
- `<LUT>` is the last update time upto which namespace or set needs to be truncated. LUT is either nanosecond since Unix epoch like `1513687224599000000` or a date string in format `"Dec 19 2017 12:40:00"`.

Example:

```sql
aql> TRUNCATE test.demo
```
