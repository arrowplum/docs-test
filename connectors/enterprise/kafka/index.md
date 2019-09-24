---
title: Overview for Aerospike Connect for Kafka
description: Aerospike Connect for Kafka connector overview
---

## Overview

As a database, Aerospike is entrusted as a core "source of truth" for data. But
it is rarely used in isolation and instead is often part of a larger system
consisting of one or more data stores, messaging systems and information
processing systems. In these distributed systems, a messaging system is often
used to connect the individual system components. Aerospike Connect for Kafka is intended
to make it easy to integrate an Aerospike database into a larger system's
architecture by supporting bi-directional data exchange between Aerospike and
Kafka. The two connectors provided support streaming data from Kafka
into an Aerospike database (“Inbound Connector”), as well as publication of
Aerospike database updates to Kafka (“Outbound Connector”).

## Outbound Connector
{{log this}}
![]({{book.assets}}/images/kafka_outbound.png)

The outbound connector provides a reliable and scalable mechanism to publish
data from an Aerospike Server (Enterprise Edition) to Kafka. It builds on the
Aerospike [Change Notification Framework](/docs/architecture/change-notification.html),
which provides the ability to subscribe to change notifications, including
record updates and deletions. It delivers the notifications via the HTTP
protocol. The connector receives these notifications, provides
capabilities to transform the proprietary Aerospike wire protocol into a number
of common data serialization formats, and then sends the updates to a Kafka
broker.

<a href="/docs/connectors/enterprise/kafka/outbound/features.html" class="primary button">Learn more</a> 

## Inbound Connector

![]({{book.assets}}/images/kafka_inbound.png)

The inbound connector supports streaming data from one or more Kafka topics and
persisting the data in an Aerospike database. It leverages the open source
Kafka Connect framework, that is part of the Apache Kafka project. In terms of
Kafka Connect, the inbound connector implements a "sink" connector. For more
information about Kafka Connect, please refer to the [Apache Kafka
documentation](https://kafka.apache.org/documentation/#connect).

<a href="/docs/connectors/enterprise/kafka/inbound/features.html" class="primary button">Learn more</a> 
