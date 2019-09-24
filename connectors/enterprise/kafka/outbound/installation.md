---
title: Install Aerospike Connect for Kafka - Outbound
description: Learn how to install Aerospike Kafka outbound connector.
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 6
---

## Prerequisites

### Java Run-time Environment

The outbound connector is a Java Spring Boot web application. It requires a
Java run-time environment. The connector requires Java 8 as a minimum
version. It supports, and has been tested against, Java 10 as well.

### Conscrypt

Transport Layer Security (TLS) is supported through the use of the Conscrypt
Java Security Provider implementation. The Conscrypt package includes a native
library with support for the following platforms:

* Linux, x86_64 (64-bit)
* Mac, x86_64 (64-bit)
* Windows, x86 (32-bit)
* Windows, x86_64 (64-bit)

## Supported Platforms

The connector has been tested on and supports the following platforms:

* RHEL / CentOS 6
* RHEL / CentOS 7
* Ubuntu 14.04+
* Ubuntu 16.04+
* Ubuntu 18.04+
* Debian 7
* Debian 8
* Debian 9

## Installation

The installation steps below apply to both the Debian (\*.deb) and RHEL
(\*.rpm) packages. Where there are significant differences between the
supported platforms, this will be pointed out.

{{#steps}}
{{#steps-step 1 "Install Java 8" markdown=true}}

The Aerospike Connect for Kafka package does not include a Java run-time environment. Most supported platforms provide official JDK 8 packages. For platforms that do not, Oracle's JDK 8 builds can be obtained from [jdk.java.net/8/](http://jdk.java.net/8/).

**RHEL, CentOS, etc.**

```
$ sudo yum install java-1.8.0-openjdk
```

**Debian, Ubuntu, etc.**

```
$ sudo apt-get install openjdk-8-jre
```
{{/steps-step}}

{{#steps-step 2 "Install connector package" markdown=true}}

Go to the [Aerospike Enterprise downloads](/enterprise/download/solutions/aerospike-kafka/) site to download the connector package for your platform and install it.

**RHEL, CentOS, etc.**

```
$ sudo rpm -i aerospike-kafka-1.0.0-1.x86_64.rpm
```

**Debian, Ubuntu, etc.**

```
$ sudo dpkg -i aerospike-kafka-1.0.0.deb
```
{{/steps-step}}

{{#steps-step 3 "Install Aerospike Enterprise feature key file" markdown=true}}

Copy the Aerospike Enterprise feature key file to
`/etc/aerospike-kafka/features.conf`. The `mesg-kafka-connector` feature
needs to be enabled in order to run the Kafka connector.

```
$ sudo cp features.conf /etc/aerospike-kafka/
```
{{/steps-step}}

{{#steps-step 4 "Verify/update connector configuration" markdown=true}}

The connectors configuration can be found in the `/etc/aerospike-kafka/` directory. For details on how to configure the Kafka bootstrap servers, enable TLS, etc., please refer to the [configuration guide](/docs/connectors/enterprise/kafka/outbound/configuration.html).

{{/steps-step}}

{{#steps-step 5 "Start the connector" markdown=true}}

The connector package includes both a Systemd service definition as well as a
System V init script. Depending on the chosen platform, the appropriate
method to start up the connector should be used:

**Systemd**

```
$ sudo systemctl start aerospike-kafka
```

**System V init**

```
$ sudo service aerospike-kafka start
```
{{/steps-step}}

{{#steps-step 6 "Verify installation" markdown=true}}

To test the package installation and configuration, send a test package to
the connector and verify that it gets published to Kafka.

First, start the `kafka-console-consumer` included in the Kafka installation:

```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic aerospike
```

Next, use the test script included in the `aerospike-kafka` package to send the test message:

```
$ cd /opt/aerospike-kafka
$ ./bin/test.sh
OK - Sent file "test/test-record.bin" to http://localhost:8080/aerospike/kafka/publish successfully.
```

If the message was sent successfully, you should see the test record printed out by the console consumer in JSON format:

```json
{"msg":"write","key":["test","tests","i2Ejrq8uPFTLpwAn2TI2YcaybfQ=","key"],"gen":0,"exp":0,"lut":0,"bins":[{"name":"i","type":"int","value":42},{"name":"s","type":"str","value":"foo"},{"name":"f","type":"float","value":1.99}]}
```
{{/steps-step}}

{{/steps}}
