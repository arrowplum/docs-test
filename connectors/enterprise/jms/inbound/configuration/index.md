---
title: Configure Aerospike Connect for JMS - Inbound
description: Configure Aerospike Connect for JMS - Inbound
---

Configuration of the JMS inbound connector is specified in YAML format. The
configuration file is located at _/etc/aerospike-jms-inbound/aerospike-jms-inbound.yml_.

## Sections

The configuration file is composed of the following sections

- <a href="/docs/connectors/enterprise/jms/inbound/configuration/topics.html">Queues & Topics </a>
- <a href="/docs/connectors/enterprise/jms/inbound/configuration/jms.html">JMS </a>
- <a href="/docs/connectors/enterprise/jms/inbound/configuration/aerospike.html">Aerospike </a>
- <a href="/docs/connectors/enterprise/jms/inbound/configuration/feature-key.html">Feature Key </a>
- <a href="/docs/connectors/enterprise/jms/inbound/configuration/logging.html">Logging </a>

## Example

A sample to stream data from IBM MQ is shown below

```
queues:
  DLQ:
    aerospike-operation:
      type: write
    parsing:
      format: json
    mapping:
      all-fields: true
      key-field: key
      namespace:
        mode: static
        value: test

jms:
  factory: com.ibm.mq.jms.MQConnectionFactory
  config:
    hostName: 192.168.50.2
    port: 1414
    queueManager: QM1
    transportType: 1
    channel: DEV.APP.SVRCONNs

feature-key-file: features.conf

aerospike:
  seeds:
    - 192.168.50.1
```
