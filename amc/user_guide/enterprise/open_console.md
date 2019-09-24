---
title: Display Console
description: Learn the requirements and methods to access the AMC Server.
---

{{#todo}}
TODO:
- Links on the Dashboard and other pages
{{/todo}}

# Requirements to Access AMC Server

You can access AMC on any workstation (Linux, Windows, Mac) as long as you have a supported browser:

Mozilla Firefox (preferred) or
Google Chrome


Depending on how you set your throughput monitoring window you will also need:

2 GB free RAM if throughput view is up to 10 minutes

4 GB free RAM if throughput view is greater than 10 minutes

Starting the Console
Once the AMC server is installed and started, open the console in your browser with the following address:

```bash
http://<AMC Server IP>:8081/
```

You will be prompted for the "Host Name or IP". Simply provide the seed node so that AMC can find your Aerospike cluster. Note that you can specify any node in the cluster. The IP address text box can take an IP address, a fully qualified domain name, or even just "localhost".  If you have changed the service port, then you must specify the correct port. When you click "Connect" the application will start and the console will display.

For versions 4.0 and above AMC allows connecting to TLS enabled secure Aerospike clusters. The certificates for the Aerospike secure cluster have to be set up in the [Configuration](/docs/amc/configure/configuration.html#tls-server-certificates) of the AMC at start up. At the time of login you need to enter the name of the certificate identifying the TLS enabled secure Aerospike cluster. This field can be left blank for non TLS Aerospike clusters.

Optionally, you can name a cluster to reconnect to it again later. In case security for Aerospike is enabled, users can access amc features depending on their roles.  An Aerospike user with no roles assigned can be used by AMC to conduct basic monitoring of an Aerospike cluster.

{{#note}}Cluster names are user session specific and not across multiple sessions.{{/note}}
   
   <table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24" style="max-width: none">
		</td>
		<td>
			The Cluster Seed Node popup will display fields to enter host IP and port. Give the cluster a name which you find suitable for the cluster. The IP address text box can take an IP address, a fully qualified domain name, or even just "localhost".
		</td>
	</tr>
    <tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24" style="max-width: none">
		</td>
		<td>
			If the Aerospike cluster has security enabled, a Log In popup will be displayed. Provide valid Username and password and click on submit.
		</td>
	</tr>
	</table>


<img src="/docs/amc/assets/images/e06_connect.png" alt="seed node" width="500">
 
<img src="/docs/amc/assets/images/e06_Login.png" alt="Login" width="500">

