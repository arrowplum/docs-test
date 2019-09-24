---
title: Install Aerospike Connect for Kafka - Inbound
description: Learn how to install Aerospike Kafka inbound connector.
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 5
---

## Prerequisites

### Kafka Connect

The Kafka Connect framework was added to Kafka in the 0.9.0.0 release. You can
install Kafka through you distro's package management system or by downloading
it from the [Apache Kafka project page](https://kafka.apache.org/downloads).

## Installation

{{#steps}}

{{#steps-step 1 "Install connector package" markdown=true}}

Go to the [Aerospike Enterprise downloads](/enterprise/download/solutions/aerospike-kafka/)
site to download the inbound connector. Both the inbound and outbound
connectors are included in the two platform specific packages (.deb / .rpm).
There are also separate .tar.gz and .zip package that only include the inbound
connector.

**RHEL, CentOS, etc.**

```
$ sudo rpm -i aerospike-kafka-1.0.0-1.x86_64.rpm
```

**Debian, Ubuntu, etc.**

```
$ sudo dpkg -i aerospike-kafka_1.0.0.deb
```

The inbound connector is installed into the `/opt/aerospike-kafka-connect-sink` directory.

{{/steps-step}}

{{#steps-step 2 "Install Aerospike Enterprise feature key file" markdown=true}}

Copy the Aerospike Enterprise feature key file to the connector machine, e.g. to
`/etc/aerospike-kafka/features.conf`. Then update the feature key path in
`/opt/aerospike-kafka-connect-sink/etc/aerospike-sink.properties`. The
`mesg-kafka-connector` feature needs to be enabled in order to run Aerospike Connect for Kafka.

Feature key path in `aerospike-sink.properties`:

```
feature_key.path=/etc/aerospike-kafka/features.conf
```
{{/steps-step}}

{{#steps-step 3 "Verify/update connector configuration" markdown=true}}

Check the Aerospike Connect for Kafka configuration
(`/opt/aerospike-kafka-connect-sink/etc/aerospike-sink.properties`) and ensure
the Aerospike cluster, Kafka topic, and policy settings are configured
correctly. For more information, please refer to the [configuration guide](/docs/connectors/enterprise/kafka/inbound/configuration.html).
{{/steps-step}}

{{#steps-step 4 "Run the connector in standalone mode for testing" markdown=true}}

In standalone mode, Kafka Connect runs the connector in a single process.

```
$ cd /opt/aerospike-kafka-connect-sink
$ path/to/kafka/bin/connect-standalone.sh etc/connect-standalone.properties etc/aerospike-sink.properties
```
{{/steps-step}}

{{#steps-step 5 "Run the connector in distributed mode for deployment" markdown=true}}

Kafka Connect distributed mode handles automatic balancing of work, allows you
to scale up (or down) dynamically, and offers fault tolerance both in the
active tasks and configuration and offset commit data.

For instructions how to run Kafka Connect in distributed mode, please consult
the [Kafka Connect user guide](https://kafka.apache.org/documentation/#connect_user).

{{/steps-step}}

{{#steps-step 6 "Verify installation" markdown=true}}

Kafka's producer script can be used to verify everything is set up correctly.

This example assumes the configuration has `topic.set=test` in the configuration. It adds an aerospike record with the key `testrecordkey` and two string bins `bin1` and `bin2`

```sh
 bin/kafka-console-producer.sh --broker-list localhost:9092  --property "key.separator=:" --property "parse.key=true" --topic test
 testrecordkey:{"bin1": "value1", "bin2": "value2"}
```

You can then query aerospike in AQL using:

```sh
aql --command="select bin1,bin2 from test.test where PK='testrecordkey'"
```

and verify your information is now in the database.

{{/steps-step}}
{{/steps}}
