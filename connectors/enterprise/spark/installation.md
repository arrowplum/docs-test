---
title: Aerospike Connect for Spark Installation
description: Learn how to deploy and configure Aerospike Connect for Spark.
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 5
---

Aerospike Connect for Spark runs on Linux, Windows and OS X and requires Java 8. Follow these steps to install and configure Aerospike Connect for Spark.

{{#steps}}
{{#steps-step 1 "Install java 8 (Linux example below)" markdown=true}}

```bash
$ wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u121-b13/e9e7ea248e2c4826b92b3f075a80e441/jdk-8u121-linux-x64.tar.gz"
$ sudo tar xzf jdk-8u121-linux-x64.tar.gz -C /opt
$ cd /opt/jdk1.8.0_121/
$ sudo alternatives --install /usr/bin/java java /opt/jdk1.8.0_121/bin/java 2
```

{{/steps-step}}
{{#steps-step 2 "Install Aerospike Connect for Spark" markdown=true}}
Go to the [Aerospike Enterprise downloads site](/enterprise/download/aerospike-spark/) to download the Aerospike Connect for Spark tgz file and extract it into the /opt directory.
```bash
$ sudo tar zxvf aerospike-spark-1.1.0.tgz -C /opt
```

Or Download Aerospike Connect Plugin jar file only for exisitng spark cluster
{{/steps-step}}
{{#steps-step 3 "Install Aerospike Enterprise Feature Key File" markdown=true}}

Install the Aerospike Enterprise [`feature-key-file`](/docs/reference/configuration#feature-key-file) on the Aerospike Connect for Spark server. The default location for the feature key file is ```/etc/aerospike/features.conf```. This location can be configured using the ```aerospike.keyPath``` configuration option.

{{/steps-step}}
{{#steps-step 4 "Set Up Environment Variables" markdown=true}}

**Aerospike Connect and Spark integrated downlaod (.tgz file)**

```
$ sudo vi /etc/profile
```

Add these lines to /etc/profile:

```
export AS_SPARK_HOME=/opt/aerospike-spark-1.1.0
PATH=$PATH:$AS_SPARK_HOME/bin
```

```
$ source /etc/profile
```
**Aerospike Connect standalone plugin (jar file)**


Refer to relavant Spark document for [`Spark submit`](https://spark.apache.org/docs/latest/submitting-applications.html), and [`library management`](https://docs.databricks.com/user-guide/libraries.html)

{{/steps-step}}
{{#steps-step 5 "Check The Installation" markdown=true}}

```
$ which spark-shell
/opt/aerospike-spark-1.1.0/bin/spark-shell
```
{{/steps-step}}
{{/steps}}

### Cluster deployment options

Aeropsike Connect for Spark supports several clustering options. Refer to the [cluster mode overview](https://spark.apache.org/docs/2.3.0/cluster-overview.html) in the Spark documentation for further details.
