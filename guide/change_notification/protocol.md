---
title: Protocol
assets: /docs/guide/assets
---

The notifications are sent in HTTP to communicate with external systems. When a
write occurs, and the digest log is updated, the shipping thread will compose an
HTTP request, and POST the change to one of a number of HTTP servers - compared
to XDR, which will use an embedded Aerospike client to write to a remote
cluster. As HTTP is a well established protocol, with a wide variety of HTTP
server implementations with different performance levels, the user can choose
his preferred web server to consume the records. The user can also choose his
preferred programming language. As most 4th generation languages come with an
in-built library for HTTP web server, it will be very easy to consume the data.
HTTP also includes strong security primitives, allowing high quality TLS
certificate authentication, as well as protocol improvements such as HTTPS v2
which greatly improve the efficiency of secure transport.

We support the following http protocols:
* HTTP v1.1
* HTTPS v1.1
* HTTP v2
* HTTPS v2

We support transparent upgrade from HTTP v1.1 to HTTP v2 if the receiving web
server supports it (via ALPN or NPN).

## Performance
In our benchmarks clear text HTTP v2 performs the best. So, we recommend it if
the source and the destination are in a private network which is secured. If the
network is not secured the preferred option is HTTP2 with SSL. If the web server
does not support HTTP2, only then HTTP v1.1 should be used.

Generating HTTP messages within Aerospike has extra cost, compared to generating
XDR messages. Use of this feature will increase CPU load on an Aerospike
cluster, and revisiting sizing of the source cluster may or may not require
additional server nodes.
