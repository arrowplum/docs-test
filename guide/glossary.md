---
title: Glossary
description: Definitions of Aerospike specific terms. 
---

The Aerospike schemaless data model gives application designers maximum flexibility. Aerospike uses the following terms to differentiate it from the relational database (RDBMS) world. In our documentation, we introduce Aerospike concepts with their corresponding common RDBMS terms.

<table>
<tbody>
<tr>
<th>Term</th>
<th>Definition</th>
</tr>
<tr>
<td style="vertical-align:top">asd</td>
<td>This Aerospike database process runs on a server or node. </td>
</tr>
<tr>
<td style="vertical-align:top">bin</td>
<td>In the Aerospike database, each _record_ (similar to a _row_ in a relational database) stores data using one or more bins (like _columns_ in a relational database). The major difference between bins and RDBMS columns is that you don't need to define a schema. Each record can have multiple bins. Bins accept these [data types](/docs/guide/data-types.html):
<ul>
<li>Integer</li>
<li>Double</li>
<li>String</li>
<li>Bytes</li>
<li>List</li>
<li>Maps</li>
<li>Geospatial</li>
</ul>
Although the bin for a given record/object must be typed, bins in different rows do not have to be the same type. There are some internal performance optimizations for single-bin namespaces.</td>
</tr>
<tr>
<td style="vertical-align:top">[client](/docs/architecture/clients.html)</td>
<td>In Aerospike documentation, client refers to the client application, and we use client, API, and application interchangeably. All operator results return to the client.</td>
</tr>
<tr>
<td style="vertical-align:top">cluster</td>
<td>Aerospike is a distributed database, made up of one or more database nodes, called a cluster. The cluster acts together to distribute and replicate both data and traffic. Client applications use Aerospike APIs to interact with the cluster, rather than with individual nodes. This means that the application does not need to know cluster configuration. As you add or remove nodes from the cluster, it dynamically adjusts without needing any application code or configuration changes.</td>
</tr>
<tr>
<td style="vertical-align:top">digest</td>
<td>Digests are a hash of the key. The keys are hashed using the RIPEMD-160 algorithm, which takes a key of any length and always returns a 20-byte digest. Use digests for long keys. For example, using a digest for a 200-byte key improves wire performance by saving 180 bytes.</td>
</tr>
<tr>
<td style="vertical-align:top">migration</td>
<td>When nodes are added or removed from a cluster, data migrates between the remaining nodes.</td>
</tr>
<tr>
<td style="vertical-align:top">namespace</td>
<td>Aerospike clusters contain one or more namespaces (RDBMS _tablespace_). Don't think of namespaces as an RDBMS table. Namespaces segregate data with different storage requirements. For example, some data may have high performance/low storage requirements more suitable for RAM, while other data can be stored on low-cost SSDs. The Aerospike schemaless data model allows you to mix data types within a namespace. You can store data on users and URLs in the same namespace, and separate them using _sets_. </td>
</tr>
<tr>
<td style="vertical-align:top">node</td>
<td>Every Aerospike cluster has 1 or more node. These are the individual servers that act together to fulfill client requests. We do not recommend putting more than one node on a hardware server. Aerospike makes full use of a server, so putting multiple nodes on a single host is not optimal.</td>
</tr>
<tr>
<td style="vertical-align:top">policy</td>
<td>Use Aerospike clients to create policies for applications to specify operation behavior in the Aerospike database.</td>
</tr>
<tr>
<td style="vertical-align:top">record/object </td>
<td>A record or object (RDBMS _row_) in an Aerospike cluster is a primary key-value pair.</td>
</tr>
<tr>
<td style="vertical-align:top">set</td>
<td> Sets are the same as RDBMS _tables_, except that you don't have to define a schema. </td>
</tr>
<tr>
<td style="vertical-align:top">[UDF](/docs/guide/udf.html)</td>
<td>A User-Defined Function (UDF) is code written by a developer that runs inside the Aerospike database server. UDFs can significantly extend the capability of the Aerospike Database engine in functionality and in performance. Aerospike currently only supports [Lua](http://lua.org) as the UDF language.</td> 
</tr>
<tr>
<td style="vertical-align:top">[XDR](/docs/architecture/xdr.html) </td>
<td>The Aerospike cross-datacenter replication (XDR) module allows clusters to synchronize over a longer delay link. XDR is asynchronous, but delay times are often under a second. Each write is logged, and that log is used to replicate data to remote clusters.  </td>
</tr>
</tbody>
</table>
