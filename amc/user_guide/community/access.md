---
title: Display Console
definition: Learn the requirements and methods to access the AMC Server.
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

You will be prompted for the "Host Name or IP". Simply provide the seed node so that AMC can find your Aerospike cluster. Note that you can specify any node in the cluster.  The IP address text box can take an IP address, a fully qualified domain name, or even just "localhost".  If you have changed the service port, then you must specify the correct port.  When you click "Connect" the application will start and the console will display.

<img src="/docs/amc/assets/images/c01_01_connect.png" alt="seed node">

<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24" style="max-width: none"> 
		</td>
		<td>
			There are a series of different pages that you may select from at the top of the page. They are (Enterprise Edition will have additional pages):
			<ul>
				<li> Dashboard</li>			
				<li> Statistics</li>			
				<li> Definitions</li>			
				<li> Jobs</li>
			</ul>
		</td>
	</tr>	
	<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="3" width="24" style="max-width: none"> 
		</td>
		<td>
			You can change the time interval that is being graphed by changing the "Snapshot for last" value.			
		</td>
	</tr>
</table>

<img src="/docs/amc/assets/images/c01_02_firstpage_1.png" alt="AMC dashboard" width="800">



