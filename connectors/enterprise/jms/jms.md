---
title: Aerospike Connect for JMS - JMS Configuration
description: Configuring Aerospike Connect JMS provider section
---

The JMS section configures the connection properties to the JMS message broker.

| Option                      | Required | Default | Description                                                                                                                                                                                                  |
| --------------------------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| factory                     | yes      |         | The name of the connection factory. Will be specific to the message broker being used.                                                                                                                       |
| jndi-cf-name                | no       |         | The name of the JNDI factory configured at the message broker.                                                                                                                                               |
| spec                        | no       | 1.0     | The JMS spec the message broker supports.                                                                                                                                                                    |
| config                      | no       |         | The configuration to be passed to the connection factory.                                                                                                                                                    |
| import-config-file          | no       |         | The configuration to import from a config file. This should be used for sensitive configuration parameters like security credentials. Will be merged with config. See <a href="#import-config"> example</a>. |
| max-connections             | no       | 1       | The maximum number of JMS connections to establish to the JMS message broker.                                                                                                                                |
| max-sessions-per-connection | no       | 1       | The maximum number of JMS sessions to open per JMS connection.                                                                                                                                               |
| credentials                 | no       |         | Credentials required to authenticate to the JMS message broker. See <a href="#credentials">credentials</a> for details.                                                                                      |

## Credentials Config

The way the security credentials are provided depend on the JMS message broker.
You could use the credentials section to provider username and password based
credentials.

| Option        | Required | Description                                                                                    |
| ------------- | -------- | ---------------------------------------------------------------------------------------------- |
| username      | yes      | The username                                                                                   |
| password-file | yes      | The file containing the password. See <a href="#password-file"> password file</a> for details. |

## Password file

A password is stored in a password file, and the file is passed as a config
option _password-file_. Password is read from this file, everything after the
first newline is ignored; trailing spaces in the first line are not ignored.

## Supported Message Brokers

By default the connector supports the following JMS message brokers

- IBM MQ
- Solace
- ActiveMQ Artemis
- RabbitMQ

To connect to a different JMS message broker ensure that the .jar files
containing the factory and its dependencies are placed in
_/opt/aerospike-jms-inbound/usr-lib_ directory.

## Examples

### IBM MQ

```
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
```

### Solace

```
jms:
  factory: com.solacesystems.jndi.SolJNDIInitialContextFactory
  jndi-cf-name: /JNDI/CF/Aerospike
  config:
    java.naming.provider.url: 10.0.1.54:55555
    java.naming.security.principal: username@jms-inbound-test
    java.naming.security.credentials: password
  import-config-file: credentials.yml
```

### ActiveMQ Artemis

```
jms:
  factory: org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory
  jndi-cf-name: ConnectionFactory
  config:
    java.naming.provider.url: tcp:/10.2.54.1:61616?user=artemis&password=simetraehcapa
    java.naming.security.principal: artemis
    java.naming.security.credentials: simetraehcapa
  max-connections: 32
  max-sessions-per-connection: 8
```

### RabbitMQ

```
jms:
  factory: com.rabbitmq.jms.admin.RMQConnectionFactory
  config:
    host: 10.0.2.34
    port: 5672
    username: guest
    password: guest
```

### Import Config

The contents of the _import-config-file_ and _config_ are merged.

- The config file

```
jms:
  factory: com.rabbitmq.jms.admin.RMQConnectionFactory
  config:
    host: 10.0.2.34
    port: 5672
  import-config-file: credentials.yml
```

- The imported credentials file

```
// credentials.yml
username: guest
password: guest
```

The contents of the _import-config-file_ are merged with _config_ and the above
configuration is equivalent to the following

```
jms:
  factory: com.rabbitmq.jms.admin.RMQConnectionFactory
  config:
    host: 10.0.2.34
    port: 5672
    username: guest
    password: guest
```
