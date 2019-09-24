---
title: Data Model
description: The Aerospike schemaless data model provides storage flexibility.
assets: /docs/guide/assets
---

The Aerospike data model does not conform to a rigid schema. Changes to data types does not require modifying a schema. You can add new bin types on the fly. Below we describe namespace, set and bin, which are the three essential concepts in Aerospike relating to data modeling and data storage.

### Namespaces

Namespaces are top-level data containers. A namespace can actually be part of a database or a group of databases, as referred to in standard RDBMS. The way you collect data in namespaces relates to how the data is stored and managed. A namespace contains records, indexes, and policies. Policies dictate namespace behavior, including:

- How data is stored: on DRAM or disk.
- How many replicas exist for a record.
- When records expire.

For more information, see [Configuring Namespaces](/docs/operations/configure/namespace).

A database can specify multiple namespaces, each with different policies to fit your application. Consider namespaces physical containers that bind data to a storage device (a RAM segment, disk, or file, or none).

<div style="float: left" >
{{#figure "" "Namespaces using different storage engines" size="large"}}
![Namespaces using different storage engines]({{book.assets}}/model_namespace_small.png)
{{/figure}}
</div>

This illustrates a database with two namespaces, *ns1* and *ns2*. *ns1* stores records on disk. *ns2* stores records in RAM.
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
### Sets

In namespaces, records belong to logical containers called *sets*. Sets allow applications to logically group records in collections. Sets inherit the policies defined by their namespace, and can define additional policies or operations specific to the set. For example, secondary indexes can be specified only on data for a particular set, or a scan operation can be done on a specific set.

{{#note}}
Set specification is optional. Some records in the namespace may not be within a set.
{{/note}}

<div style="float: left" >
{{#figure "" "*ns1* namespace contains sets *people* and *place*, and records not belonging to any set"}}
![]({{book.assets}}/model_set_small.png)
{{/figure}}
</div>

This shows two sets, *people* and *places*, that belong to the *ns1* namespace, which also contains records not within a set. 
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
### Records

The Aerospike Database is a row store with focus on individual records (RDBMS *rows*). A record is the basic unit of storage in the database. Records can belong to a namespace or to a set within the namespace. Records use a key as their unique identifier.

Records comprise the following:

| Component     | Description |
| ---           | --- |
| key           | Unique identifier. Records are addressable using a hash of its key, called the *digest*. |
| metadata      | Record version information (generation) and the configured expiration, called the *time-to-live* (TTL). |
| bins          | Bins are equivalent to fields in RDBMS. |

#### Keys and Digests

Use keys to read or write records in your application. When a key is sent to the database, it and its set information is hashed into a 160-bit digest, which is used to address that record for all operations. So, you use the key in your application, while the digest is used for addressing the record in the database.

{{#note}}
Integers, strings, and byte are allowable data types (see *[Data Types](/docs/guide/data-types.html)*).
{{/note}}

#### Metadata

Each record contains the following metadata:

- **generation** tracks record modification cycle. The number is returned to the application on reads, which can use it to ensure that the data being written has not been modified since the last read.
- **time-to-live (TTL)** specifies record expiration. Aerospike automatically expires records based on their TTL. The TTL increments every time the record is written. For server version 3.10.1 and above, the client can set a policy to not update the TTL when updating the record. See respective client API docs for details.
- **last-update-time (LUT)** specifies the timestamp record was updated. This is a metadata internal to the database and is not returned to the client.

#### Bins

Within a record, data is stored in one or many bins. Bins consist of a name and a value. Bins do not specify the data type. Data type is defined by the value contained in the bin. This dynamic data typing provides flexibility in the data model. For example, a record can contain the bin *id* comprised of the string value *bob*. The value of the bin can always change to a different string value, but it can also change to values of a different data type, such as integer.

Records in a namespace or set can be composed of very different bins. There is no schema for the records, so each record can have a varied set of bins. You can add and remove bins at any point in the lifetime of a record. Bin values can be any [native supported data type](/docs/guide/data-types.html). 

{{#note}}
Currently, the number of concurrent unique bin names in a namespace is limited to 32K. This is due to an optimized string-table implementation.
{{/note}}
