---
title: Features of Aerospike Connect for Kafka - Inbound
description: Features of the Aerospike Connect for Kafka Inbound Connector
---

## Supported Database Operations

Via the inbound connector, new database records can be inserted into an
Aerospike cluster, and existing records can be updated. Deleting records via
the inbound connector is currently not supported.

## Standalone vs. Distributed Mode

The Aerospike Connect for Kafka inbound connector is implemented as a "sink" connector for
the Kafka Connect framework.

Aerospike Connect for Kafka supports two modes of execution:

1. standalone (single process)
1. distributed

In standalone mode all work is performed in a single process. This
configuration is simpler to setup and get started with, but it does not benefit
from some of the features such as fault tolerance. Distributed
mode handles automatic balancing of work, supports automatic scaling, and offers
fault tolerance both in the active tasks and for configuration and offset commit
data. For more information, please refer to the [Kafka Connect
documentation](https://kafka.apache.org/documentation/#connect).

## Record Mapping

The connector supports a simple, but flexible mapping of Kafka topics and
records into Aerospike namespaces and sets. The Aerospike namespace and set
can be configured per Kafka topic or the set name can be extracted from one of
the fields of the Kafka record. The key of the Aerospike record is based on the
key of the Kafka record. For Kafka records without a key, the Aerospike key can
be extracted from one of the fields of the Kafka record. The connector can
either copy the entire Kafka record or it can selectively copy individual
record fields. The Kafka record fields can be mapped to Aerospike bin names,
e.g. to avoid the length restrictions on Aerospike bin names.

For more details, please refer to the [configuration
section](/docs/connectors/enterprise/kafka/inbound/configuration.html).

<a href="/docs/connectors/enterprise/kafka/inbound/installation.html" class="primary button">Install Connector</a> 
