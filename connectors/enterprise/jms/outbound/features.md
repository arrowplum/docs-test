---
title: Features of Aerospike Connect for JMS - Outbound
description: Features of Aerospike Connect for JMS Outbound Connector
---

## Supported Database Operations

The outbound connector receives data from the Aerospike server via the [Change
Notification Framework](/docs/architecture/change-notification.html) and 
publishes the change to a JMS based message broker. Each message contains either
an updated/inserted record or a record deletion notification.  Each update/insert
message contains the full database record including all or a subset of the
record's bins. Operations on CDT List or Map types are not supported, e.g. adding a list element or removing a map entry. In
case of partial bin updates such as these, the full record bin will be
retransmitted.  Record deletion notifications contain the record digest but not
the record bins.

## Message Routing

The connector can route incoming messages to JMS queues/topics. The
routing can be static or dynamic depending on some property of the message
itself. The connector supports the following routing modes:


 * **Static:** Always route to a static queue/topic.
 * **Namespace:** Use the namespace of the Aerospike record as the JMS queue/topic.
 * **Set:** Use the set of the Aerospike record as the JMS queue/topic.
 * **Bin:** Sets the route based on the value of a bin in the record. Only string,
    blob and integer bin types are supported.
 * **Path:** Sets the route based on the URL path used to submit the record. The URL is configured in Aerospike's JMS XDR data center section.

                                                   

## JMS Outbound Message Format

The connector consumes the records from the change notification framework and
transforms them before writing to the JMS queue/topic for easier consumption by 
the downstream applications. The following message serialization formats, shared
with [Kafka Outbound Connector](/docs/connectors/enterprise/kafka/outbound/features.html)
, are currently supported:

* JSON
* MessagePack

JSON (JavaScript Object Notation) is a light-weight data serialization format.
It is a text-based format that is easy for humans to read and write, as well as
easy for computers to parse and generate. JSON implementations exist for a wide
variety of popular programming languages. For more details, please refer to the
[JSON Serialization Format specification](/docs/connectors/enterprise/kafka/outbound/json-serialization-format.html).

MessagePack is a binary data serialization format. Its compact and simple form
is designed for efficient transmission of data over the wire. MessagePack
parser implementations exist for a wide variety of popular programming
languages. For more details, please refer to the [MessagePack Serialization
Format specification](/docs/connectors/enterprise/kafka/outbound/messagepack-serialization-format.html).

<a href="/docs/connectors/enterprise/jms/outbound/installation.html" class="primary button">Install Connector</a> 
