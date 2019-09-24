---
title: Development Overview
description: Develop with serveral different clients. Depending on the requirements, different clients can be used such as Java, C, C#, Nodejs, Python, Go and more. 
---

Aerospike offers a number of client Libraries.

Please see [Client Feature Matrix](/docs/guide/client_matrix.html) for feature parity in each client.

If you are familiar with Java code, we strongly suggest that your first step is to review the [First Programs Guide](/docs/client/java/first_program.html) which includes the source code for two complete programs along with a description of the code; one program will populate a large number of records in an Aerospike DB and another that will randomly read a large number of records.

##High Performing Clients Supported by Aerospike

These clients are highly recommended for main application development as they offer the highest performance and support the latest features of the Aerospike database.

### [Java](/docs/client/java)
Aerospike’s Java client enables you to build applications in Java that store and retrieve data from an Aerospike cluster. It contains both synchronous and asynchronous calls to the database.

<a href="/docs/client/java" class="primary button">Learn more</a>

### [C#](/docs/client/csharp)
Aerospike’s C# Client enables you to build applications using C# language in order to store and retrieve data from an Aerospike cluster.

<a href="/docs/client/csharp" class="primary button">Learn more</a>

### [C](/docs/client/c)
Aerospike’s C client allows you to build applications in C to store and retrieve data from an Aerospike cluster. The C client is a smart client which periodically pings nodes for the current state of the cluster and manages interactions with the Aerospike cluster.

<a href="/docs/client/c" class="primary button">Learn more</a>

### [Go](/docs/client/go)
The Go client enables you to build applications in Go that store and retrieve data from an Aerospike cluster.

<a href="/docs/client/go" class="primary button">Learn more</a>

### [REST](/docs/client/rest)
The REST client enables you to communicate with an Aerospike cluster via
RESTful requests.

<a href="/docs/client/rest" class="primary button">Learn more</a>

##Other Clients Supported by Aerospike

These clients are recommended for non-core applications such as monitoring as they may not support all the latest features of the Aerospike database. In adddition, some of these clients may not be as high performing as the primary usage clients.

### [Python](/docs/client/python)
The Python client enables you to build an application in Python with an Aerospike cluster as its database. *This client fully supports all features of the Aerospike database.*

<a href="/docs/client/python" class="primary button">Learn more</a>

### [Node.js](/docs/client/nodejs)
The Node.js client allows you to build applications in Node.js that store and retrieve data from an Aerospike cluster.

<a href="/docs/client/nodejs" class="primary button">Learn more</a>

##Community Supported Clients

These clients are community supported, with Aerospike's guidance.

### [PHP](/docs/client/php)
The PHP client enables you to build applications in PHP that store and retrieve data from an Aerospike cluster.

<a href="/docs/client/php" class="primary button">Learn more</a>

### [Rust](/docs/client/rust)
The Rust client enables you to build applications in Rust that store and retrieve data from an Aerospike cluster.

<a href="/docs/client/rust" class="primary button">Learn more</a>

### [libevent](/docs/client/libevent)
The libevent client provides an asynchronous, non-blocking API to store and retrieve data from an Aerospike database cluster.

<a href="/docs/client/libevent" class="primary button">Learn more</a>

The libevent client is written in C. It uses the libevent library explicitly, and is intended for use only by applications that already run using libevent. The application is responsible for managing the libevent event bases, and the client must be told which event base to use for each database transaction.

### [Node.js on Windows](https://github.com/aerospike/aerospike-client-nodejs)
The Node.js client port to Windows is a community supported project and suitable for application prototyping and development.

<a href="https://github.com/aerospike/aerospike-client-nodejs" class="primary button">Learn more</a>

