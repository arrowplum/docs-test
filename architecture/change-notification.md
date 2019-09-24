---
title: Change Notification Framework
assets: /docs/architecture/assets
---

As a database, Aerospike is entrusted as a core "source of truth" for data.
Besides responding to queries, it is very advantageous to notify external
systems when changes occur - called in some systems the ability to subscribe
to changes. Aerospike's new Change Notification framework allows Aerospike
servers to efficiently notify external agents of changes, and creates an
easy to build yet reliable and scalable system to do complex event processing
(CEP), inject those notifications into message queues, or insert them into
other databases.

This change notification will be correct regardless of cluster changes, and uses
the core XDR shipping system which has been a component within Aerospike for
many years. In order to understand the mechanisms by which changes are logged,
hot keys are optimized, and retransmits are handled, refer to the [XDR
architectural](/docs/architecture/xdr.html) documentation.

For ease of consumption, the Aerospike Change Notification framework will
notify external servers over the HTTP protocol. A parsing library is provided
to consume the notifications and build custom applications. Refer to the
[development guide](/docs/guide/change_notification/index.html) to understand
how to use the parsing library. [Aerospike Connect for Kafka](/docs/solutions/aerospike_connect_kafka/index.html)
is provided, thus allowing notifications to be placed in a Kafka queue for broad
distribution.

{{#figure "" "Change Notification Framework" size="large"}}
![]({{book.assets}}/change_notification.png)
{{/figure}}

# Architecture

## Source
Aerospike's shipping system for change notification is the same as XDR, and will
evolve as XDR evolves. The XDR system is configured with endpoints, and whenever
a write is applied to a configured namespace, an entry is made in the
corresponding XDR digestlog. The server which was master at time of the write is
responsible for later reading the record, and posting that record to relevant
destination data centers. As with current XDR, the system for detecting failed
links, retransmitting requests is the same.

In the current system, a change notification is the entire new version of the
record, as well as metadata about the record. In some cases, only changed bins
(columns) may be shipped, with unchanged bins not being shipped. If there is a
backlog of data to be shipped, some record versions may be skipped over in the
interest of efficiency. Compression may also be applied.

## HTTP Protocol
The shipping thread will compose an HTTP request, and uses the POST method to notify
one of the configured HTTP servers (in a load balanced way). As HTTP is a well
established protocol, with a wide variety of HTTP server implementations with
different performance levels, users can choose their preferred web server to
consume the records. User can also choose their preferred programming
language. As most 4th generation languages come with an in-built library for
HTTP web server, it should be very easy to consume the data. HTTP also includes
strong security primitives, allowing high quality TLS certificate
authentication, as well as protocol improvements such as HTTPS v2 which greatly
improve the efficiency of secure transport.

The following HTTP Protocols are supported:
* HTTP v1.1
* HTTPS v1.1
* HTTP v2
* HTTPS v2

Automatic switch from HTTP v1.1 to HTTP v2 is supported if the receiving web
server supports it (via ALPN or NPN).

## Receiver

The receiver which will be a web-server should handle two operations:
1. POST request
1. GET request

The POST method handler will consume the record sent in the change notification.
The GET method handler just responds with OK. This response will be used to
monitor the health of the webserver.

Aerospike provides a Java library to read the records coming in the change
notification. Aerospike also provides an end-to-end solution to read the
records coming in the change notification and write them to a kafka queue. Once
the records are written into kafka queue, it can be forwarded to any downstream
system. Read more about it in the [Aerospike Connect for Kafka](/docs/solutions/aerospike_connect_kafka/index.html)
solution page.
