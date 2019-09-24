---
title: Java Client Servlets Example
description: Utilize the Servlets Example using Java Client API and Aerospike database. 
---

The _servlets_ package contains source code that demonstrates how to access the Aerospike database through an HTTP interface (REST). You can import the source code into your IDE and/or build using Maven. The servlet was tested under Tomcat 6. 

The servlet files are placed in the _servlets_ folder during SDK installation.

To build the servlet:
 
```bash
$ cd servlets
$ mvn package
```

The WAR file location is: `servlets/target/aerospike.war`.

The example web page performs AJAX requests using the servlet:

```bash
http://<web host>:<web port>/aerospike/example.html
```

The servlet provides a REST interface, where keys in the Aerospike database are expressed as **namespace/set name/key name** triples.

To get or set a record, POST to the same URL, as described below.

### Write a Record

To use a URL to write a record using REST: 

```bash
http://<web host>:<web port>/aerospike/client/<namespace>/<set name>/<key>?host=<aerospike host>&port=<aerospike port>&bin=<bin name1>&value=<bin value1>&bin=<bin name2>&value=<bin value2>...
```

### Read a Record

To use a URL to read a record:

```bash
http://<web host>:<web port>/aerospike/client/<namespace>/<set name>/<key>?host=<aerospike host>&port=<aerospike port>
```

The response returns in JSON format:

```
{ "result" : {<binname>:<value>, ... <binname>:<value>}, "generation":<generation>}
```

