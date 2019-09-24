---
title: Install Aerospike Connect for JMS - Outbound
description: Learn how to install Aerospike JMS outbound connector.
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 6
---

## Prerequisites

### Java Runtime Environment

The JMS outbound connector is a Java web application requiring **Java 8** or
higher. Java 11 or higner is required if TLSv1.3 is desired. Both Oracle and
OpenJDK Java Runtime Environments are supported.

## Supported Platforms

The connector has been tested on and supports the following platforms:

* RHEL / CentOS 7
* Ubuntu 16.04+
* Ubuntu 18.04+
* Debian 8
* Debian 9
* Debian 10

## Installation

The installation steps below apply to both the Debian (\*.deb) and RHEL
(\*.rpm) packages. Where there are significant differences between the
supported platforms, this will be pointed out.

{{#steps}}
{{#steps-step 1 "Install Java 8" markdown=true}}

The Aerospike Connect for JMS package does not include a Java runtime
environment. Most supported platforms provide official JDK 8 packages. For
platforms that do not, Oracle's JDK 8 builds can be obtained from
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

{{#steps-step 2 "Install connector package" markdown=true}}

Go to the [Aerospike Enterprise downloads](/enterprise/download/solutions/aerospike-jms/)
site to download the connector package for your platform and install it.


**RHEL, CentOS, etc.**

```
$ sudo rpm -i aerospike-jms-outbound-1.0.0-1.noarch.rpm
```

**Debian, Ubuntu, etc.**

```
$ sudo dpkg -i aerospike-jms-outbound-1.0.0.all.deb
```

{{/steps-step}}

{{#steps-step 3 "Verify/update connector configuration" markdown=true}}

The connectors configuration can be found in the `/etc/aerospike-jms-outbound/`
directory. For details on how to configure the JMS provider, enable
TLS, etc., please refer to the [configuration guide](/docs/connectors/enterprise/jms/outbound/configuration/index.html).

{{/steps-step}}

{{#steps-step 4 "Start the connector" markdown=true}}

The connector package includes a Systemd service definition. The installation
procedure creates a aerospike-jms-outbound service.


Enable the connector to start on system startup or reboot.
```
$ sudo systemctl enable aerospike-jms-outbound
```

To start the connector service run.
```
$ sudo systemctl start aerospike-jms-outbound
```

{{/steps-step}}

{{/steps}}

<a href="/docs/connectors/enterprise/jms/outbound/configuration/index.html" class="primary button">Configure Connector</a> 
