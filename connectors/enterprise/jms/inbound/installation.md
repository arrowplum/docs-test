---
title: Install Aerospike Connect for JMS - Inbound
description: Learn how to install Aerospike JMS Inbound connector.
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 5
---

## Prerequisites

### Java Run-time Environment

The JMS inbound connector is a Java application. It requires a Java run-time
environment. The connector requires Java 8 as a minimum version.

## Supported Platforms

The connector has been tested on and supports the following platforms:

- RHEL / CentOS 7
- Ubuntu 16.04+
- Ubuntu 18.04+
- Debian 8
- Debian 9
- Debian 10

## Installation

The installation steps below apply to both the Debian (\*.deb) and RHEL
(\*.rpm) packages. Where there are significant differences between the
supported platforms, this will be pointed out.

{{#steps}}
{{#steps-step 1 "Install Java 8" markdown=true}}

The Aerospike Connect for JMS Inbound package does not include a Java run-time
environment. Most supported platforms provide official JDK 8 packages.
For platforms that do not, Oracle's JDK 8 builds can be obtained from
[jdk.java.net/8/](http://jdk.java.net/8/).

**RHEL, CentOS, etc.**

```
$ sudo yum install java-1.8.0-openjdk
```

**Debian, Ubuntu, etc.**

```
$ sudo apt-get install openjdk-8-jre
```

{{/steps-step}}

{{#steps-step 2 "Install JMS Inbound Connector Package" markdown=true}}

Go to the [Aerospike Enterprise downloads](/enterprise/download/solutions/aerospike-inbound/)
site to download the connector package for your platform and install it.

**RHEL, CentOS, etc.**

```
$ sudo rpm -i aerospike-jms-inbound-1.0.0-1.noarch.rpm
```

**Debian, Ubuntu, etc.**

```
$ sudo dpkg -i aerospike-jms-inbound-1.0.0.all.deb
```

{{/steps-step}}

{{#steps-step 3 "Install Aerospike Enterprise feature key file" markdown=true}}

Download the feature key file and configure the feature-key property in the
configuration file to the the location of the feature key file. The
`mesg-kafka-connector` feature should be enabled in the feature key file.

{{/steps-step}}

{{#steps-step 4 "Configure the JMS Inbound Connector" markdown=true}}

Setup the configuration of the connector in _/etc/aerospike-jms-inbound/aerospike-jms-inbound.yml_.

{{/steps-step}}

{{#steps-step 5 "Start the JMS Inbound Connector" markdown=true}}

The connector package includes a Systemd service definition.

```
$ sudo systemctl start aerospike-jms-inbound
```

{{/steps-step}}

{{/steps}}

<a href="/docs/connectors/enterprise/jms/inbound/configuration.html" class="primary button">Configure JMS Inbound</a>
