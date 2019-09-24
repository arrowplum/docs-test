---
title: Troubleshoot AMC
description: Learn how to troubleshoot the Aerospike Management Console.
---

Here are common issues with running the AMC.

#### The AMC can't connect to the cluster
If you get an error stating that the AMC cannot connect to the seed node, the most common reason for this is that there is a firewall or some other network issue between the server that is hosting the AMC and the cluster. The AMC host must be able to connect with all nodes via the service port specified. 

If the AMC interface appears to be unresponsive, the problem may be because the node(s) have multiple IP addresses and are reporting private IP addresses that the AMC cannot connect to. In order to fix this, you must use the `access-address` parameter in the configuration file of the nodes in the cluster. You must specify this in the service sub-context of the network context of the configuration file.

```ruby
service {
	...
}

network {
	service {
		...
		access-address <ip_address>
	}
}
...
```
The IP address specified should match the one accessible to the AMC host.

#### The throughput chart sometimes shows 0 throughput, but I think the server is actually serving traffic at those times
This can sometimes happen when your computer (laptop or desktop) has intermittent connectivity with the cluster. Although harmless, this can be confusing. Improving the network connection is the best way to fix this.

#### Cannot start/stop nodes
This feature is only supported in the AMC Enterprise Edition. If you are unable to start/stop nodes, this is most likely to be due to a problem with the installation of [Fabric](http://www.fabfile.org/). Check with the installation on the AMC host to make sure it has been properly configured.

