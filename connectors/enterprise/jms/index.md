---
title: Overview for Aerospike Connect for JMS
description: Aerospike Connect for JMS overview
---

## Overview

As a database, Aerospike is entrusted as a core "source of truth" for data. But
it is rarely used in isolation and instead is often part of a larger system
consisting of one or more data stores, messaging systems and information
processing systems. In these distributed systems, a messaging system is often
used to connect the individual system components. Aerospike Connect for JMS is intended
to make it easy to integrate an Aerospike database into a larger system's
architecture by supporting bi-directional data exchange between Aerospike and a
JMS message broker. The two connectors provided support streaming data 
using a JMS provider into an Aerospike database (“Inbound Connector”), as well as publication of
Aerospike database updates to a JMS message broker (“Outbound Connector”).

## Outbound Connector
{{log this}}
![]({{book.assets}}/images/jms_outbound.png)

The outbound connector provides a reliable and scalable mechanism to publish
data from an Aerospike Server (Enterprise Edition) to a JMS message broker. It builds on the
Aerospike [Change Notification Framework](/docs/architecture/change-notification.html),
which provides the ability to subscribe to change notifications, including
record updates and deletions. It delivers the notifications via the HTTP
protocol. The connector receives these notifications, provides
capabilities to transform the proprietary Aerospike wire protocol into a number
of common data serialization formats, and then sends the updates to a JMS message
broker.

<a href="/docs/connectors/enterprise/jms/outbound/features.html" class="primary button">Learn more</a> 

## Inbound Connector

![]({{book.assets}}/images/jms_inbound.png)

The inbound connector supports streaming data from one or more JMS queues/topics 
and persisting the data in an Aerospike database. The inbound JMS connector 
implements a scalable JMS consumer that can listen to updates or deletes on one or 
more topics/queues and applies them to Aerospike.

<a href="/docs/connectors/enterprise/jms/inbound/features.html" class="primary button">Learn more</a> 
