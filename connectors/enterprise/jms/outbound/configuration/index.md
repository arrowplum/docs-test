---
title: Configure Aerospike Connect for JMS - Outbound
description: Configuring the Aerospike Connect for JMS Outbound Connector
---

## Aerospike database setup
Aerospike JMS outbound connector requires configuring Aerospike database to push
updates and record deletes to the connector.

The Aerospike database configuration is described [here](/docs/connectors/enterprise/jms/outbound/configuration/aerospike.html).

# JMS Outbound Connector configuration

The outbound connector is configured via the
`/etc/aerospike-jms-outbound/aerospike-jms-outbound.yml` YAML formatted
configuration file.

A sample configuration for connecting to IBM MQ is shown below:

```
service:
  port: 8080

format: json

delivery-mode: persistent

routing:
  mode: bin
  type: queue
  bin: category
  default: test-queue
  transforms:
    - trim
    - regex:
        pattern: '[^A-Za-z0-9]'
        replacement: '-'
    - lowercase

jms:
  factory: com.ibm.mq.jms.MQConnectionFactory
  spec: 2.0
  config:
    hostName: 10.0.2.5
    port: 1414
    queueManager: QM1
    transportType: 1
    channel: DEV.APP.SVRCONN
  credentials:
    username: admin
    password-file: /path/to/password/file.txt

logging:
  file: /var/log/aerospike-jms-outbound/aerospike-jms-outbound.log
```

The configuration file has the following sections:
 * [service](/docs/connectors/enterprise/jms/outbound/configuration/service.html) - configures the connector's listening ports, TLS and network interface.
 * [format](/docs/connectors/enterprise/jms/outbound/configuration/format.html) - specifies the message format to use for the outbound messages sent
   to the message broker.
 * [delivery-mode](/docs/connectors/enterprise/jms/outbound/configuration/delivery-mode.html) - chooses between persistent and non-persistent delivery
   of messages to the message broker.
 * [routing](/docs/connectors/enterprise/jms/outbound/configuration/routing.html) - configures how incoming record updates/deletes from Aerospike are
   routed to the message broker.
 * [jms](/docs/connectors/enterprise/jms/outbound/configuration/jms.html) - configures connection properties for the target JMS message broker.
 * [logging](/docs/connectors/enterprise/jms/outbound/configuration/logging.html) - configures the destination and level for the connectors logs.
