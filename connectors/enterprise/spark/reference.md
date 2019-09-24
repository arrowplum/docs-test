---
title: Aerospike Connect for Spark Configuration Reference
description: Learn how to configure Aerospike Connect for Spark.
---

This section provides a reference for all the API extensions and configuration parameters for Aerospike Connect for Spark.

### Dataset extensions

#### aeroJoin

The ```aeroJoin[A]``` function extends the Dataset class to allow the user to load data from an Aeropsike database using batch read calls. The ```aeroJoin[A]``` function accepts the column name of the primary key and the set name. The properties from the ```A``` class determine which bins will be loaded from the Aerospike database. It returns a typed ```Dataset[A]``` where ```A``` is a user defined class:

```scala
aeroJoin[A](keyCol: String, set: String): Dataset[A]
```

#### aeroIntersect

The ```aeroIntersect``` function extends the Dataset class to allow the user to filter records from the Dataset which do not exist in the Aerospike database. The user must specify the column in the Dataset that contains the primary key of the records and the name of the Aerospike set in which to perform the intersect. The ```aeroIntersect``` call will return a Dataset which is a subset of the initial Dataset.

```scala
aeroIntersect(keyCol: String, set: String): Dataset
```

### Configuration Options

Option|Description|Default value
------|-----------|-------------
aerospike.seedhost|A host name or address of the cluster| "localhost"
aerospike.port|Service port of the Aerospike cluster| 3000
aerospike.maxthreadcount|Maximum number of threads to use for writing data to Aerospike|15
aerospike.timeout|Timeout for all operations in milliseconds|1000
aerospike.sockettimeout|Server side socket timeout for query/scan operations in milliseconds (0 = no timeout)|0
aerospike.sendKey|If true, store the value of the primary key|false
aerospike.commitLevel|Consistency guarantee when committing a transaction on the server|CommitLevel.COMMIT_ALL
aerospike.generationPolicy|How to handle record writes based on record generation|GenerationPolicy.NONE
aerospike.namespace|Aerospike Namespace|"test"
aerospike.set|Aerospike Set|no default
aerospike.updateByKey|This option specifies that updates are done by key with the value in the column specified ```option("aerospike.updateByKey", "key")```|
aerospike.updateByDigest|This option specifies that updates are done by digest with the value in the column specified ```option("aerospike.updateByDigest", "Digest")```|
aerospike.schema.scan|The number of records to scan to infer schema|100
aerospike.keyColumn|The name of the key column in the Data Frame|"__key"
aerospike.digestColumn|The name of the digest column in the Data Frame|"__digest"
aerospike.expiryColumn|The name of the expiry column in the Data Frame|"__expiry"
aerospike.generationColumn|The name of the generation column in the Data Frame|"__generation"
aerospike.ttlColumn|The name of the TTL column in the Data Frame|"__ttl"
aerospike.savemode|Write policy to use when saving records|"ignore"
aerospike.batchMax|Maximum number of records per batch read request for aeroJoin and aeroIntersect|5000
aerospike.keyPath|Path to aerospike license feature key file|/etc/aerospike/features.conf

#### Save mode (used by ```aerospike.savemode``` configuration)

Save mode| Record Exists Policy
---------|---------------------
ErrorIfExists|CREATE_ONLY
Ignore|CREATE_ONLY
Overwrite|REPLACE
Append|UPDATE_ONLY

