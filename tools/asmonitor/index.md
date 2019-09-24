---
title: Aerospike Monitor - asmonitor
description: Learn about the Aerospike monitoring tool, asmonitor, the place to start when monitoring performance, tuning configuration settings and diagnosing issues with your cluster.

---

{{#note}}
Deprecated from Aerospike Tools Versions 3.9. Use [Aerospike Admin](/docs/tools/asadm) (asadm) instead.
{{/note}}

The  Aerospike monitoring tool asmonitor is the place to start when monitoring performance, tuning configuration settings and diagnosing issues with your cluster. These pages describe how to run asmonitor and the various commands that you can use from the asmonitor console to monitor your Aerospike Database.

asmonitor (installed in /opt/aerospike/bin/asmonitor) gets the server/cluster statistics and presents them in an easy to read format.  asmonitor also allows you to dynamically change the configuration parameters for the cluster or particular nodes. The output from asmonitor can be customized to display different values.

The asmonitor tool is a console where you can type a variety of commands (including most of the -v strings that work in the asinfo command).

Usage
You can run asmonitor from any machine that has access to the network. To run asmonitor use the command:

```
asmonitor [-h <host>[:<port>]] [-p <port>]  [-c <config>]
```

The -h option specifies a hostnames or IP addresses to inquire stats from. The values for the -h option are: 

The <host> parameter is the hostname of the server. 
The <port> parameter is optional, and will default to the port specified by the -p option if not specified.
The -p option specifies the default port for the hosts being inquired, if no port is specified in the -h option. The -p is optional, and if not specified then it will default to 3000.

The first time you use asmonitor you must specify a seed node using the -h option. If -h is not specified, then it will default to localhost. Once connected to the node, then the other nodes in the cluster will be automatically discovered.  Subsequent invocations of asmonitor do not require the seed node because the configuration is saved in the configuration file.

Configuration
asmonitor manages a configuration files located at ~/.asmonitor/asmonitor.conf. The configuration files is created asmonitor after its first invocation.  

The configuration file looks like this:

```ini
[main]
  hosts = <host>[:<port>][,...]
  namespaces = <namespace>[,...]List of namespaces (auto generated. user should not edit)
  crawl = (True | False) True Find all nodes in cluster- Works only with internal IPs
  xdr = False. Disabled by default. To enable'xdr=True'
  xdrport=xdr monitor port
```

The hosts parameter specifies a comma-separated list of hostnames or IP addresses for asmonitor to connect to by default.

The <host> parameter is the hostname of the server. 
The <port> parameter is optional, and will default to the port specified by the -p option if not specified.
The namespaces parameter specifies the namespaces to be used by default.

The crawl parameter specifies whether to find all nodes in the cluster.

The xdr parameter specifies whether XDR is used by the cluster.

The xdrport parameter specifies the monitoring port for XDR. 





