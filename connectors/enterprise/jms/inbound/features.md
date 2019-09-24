---
title: Features of Aerospike Connect for JMS - Inbound
description: Features of Aerospike Connect for JMS Inbound Connector
---

The Aerospike JMS Inbound Connector streams data from a JMS message broker and
persists to an Aerospike database. Each JMS message is converted to an Aerospike
record and persisted to a Aerospike cluster.

## Limitations

- An instance of a connector can only consume from a single message broker,
  although it can consume from multiple queues/topics on the message broker.
- An instance of a connector can only persist data to a single Aerospike cluster.

## Ordering Guarantees

- Messages successfully persisted in Aerospike will be acknowledged to the
  message broker.
- Failure to persist to Aerospike will not be acknowledged. Retries of
  unacknowledged messages depends on the configuration of the message broker.
- Messages with parsing or conversion to Aerospike record errors will be logged
  and acknowledged to the message broker, since furthe retries will always
  result in parsing error. Ex: the key field could be missing in the message.
- There might be instances where messages are re-ordered on the network or
  across jms-inbound processes, in this case an older version may overwrite a
  newer version of a record in Aerospike.
- There might be instances where a record written into Aerospike might not be
  acknowledged to the messaging system, based on the retry setup of the message
  broker a record maybe written more than once into Aerospike.

## Supported Message Formats

The following message serialization formats are currently supported:

- JSON
- MessagePack
- JMS MapMessage
- Java serialized JMS ObjectMessage

<a href="/docs/connectors/enterprise/jms/inbound/installation.html" class="primary button">Install JMS Inbound</a>
