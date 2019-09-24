---
title: Namespace Detail Dashboard using the Aerospike Monitoring Console
description: Use the Aerospike Monitoring Console (AMC) to retrieve namespace details such as expired objects, evicted objects and disk usage.
---
{{#todo}}
Need a link to a description of the variables.
{{/todo}}


In order to see more detail on a particular node:
You can find a description of the variables on the [statistics and variables page](/docs/amc/user_guide/community/statistics_namespaces.html).

<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24">
		</td>
		<td>
			Go to the "Namespaces" panel.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24"> 
		</td>
		<td>
			Click on the "View Details" link for the node of interest.
		</td>
	</tr>
</table>


<img src="/docs/amc/assets/images/c01_02_namespace_detail_A_1.png" alt="node detail" width="800">

This will then open the detail page for that namespace:

<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24">
		</td>
		<td>
			A window will open up the detail for the namespace.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24"> 
		</td>
		<td>
			"Expired Objects" is the count of the number of objects that have exceeded their time-to-live (TTL). This counter continues to increment until the server has been restarted.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/5.png" alt="5" width="24"> 
		</td>
		<td>
			"Evicted Objects" is the count of the number of objects that have been deleted due to space constraints. This counter continues to increment until the server has been restarted.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/6.png" alt="6" width="24"> 
		</td>
		<td>
		Disk usage. This graphically shows not only the amount of disk used but also the level of the high water mark (HWM) and the stop write level. When the disk usage hits the HWM, the database will begin to evict (delete) data from the database. However, it is possible that the data is continually added so quickly that it may hit the stop write level. At this point the database will effectively be read only. 
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/7.png" alt="7" width="24"> 
		</td>
		<td>
			Similar to RAM.
		</td>
	</tr>
</table>


<img src="/docs/amc/assets/images/c01_02_namespace_detail_B_1.png" alt="node detail" width="800">

