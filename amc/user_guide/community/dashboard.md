---
title: Dashboard Description
description: Learn what information is displayed on the AMC dashboard.
---

# AMC Dashboard

<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24" style="max-width: none">
		</td>
		<td>
		Cluster Disk Usage:  This displays the overall disk space, with the amount of space used marked in blue.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24" style="max-width: none"> 
		</td>
		<td>
			Cluster RAM Usage: This displays the overall disk space, with the amount of space used marked in blue.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24" style="max-width: none"> 
		</td>
		<td>
			Cluster Summary: Shows the total number of nodes in the cluster and the number that are up (in green) and down (in red)
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24" style="max-width: none"> 
		</td>
		<td>
			Cluster Throughput: Shows the total number of requests (read and write) for each node. The total and successful no of transactions are the top of each graph, In graph by default you can see only clusterwide transactions,If you want to see node wise transactions then there is switch button to change to node wise details.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/5.png" alt="5" width="24" style="max-width: none">
		</td>
		<td>
Nodes: Important information about each node. 
<ul>
<li> "Cluster Visibility" means that this cluster agrees with the other nodes on the current state of the cluster. 
<li> "Replicated Objects" is the total number of objects (primary + secondary copies) 
<li> "Migrates Incoming" is the number of data partitions currently incoming from other nodes. A non-zero value means the cluster is in a dynamic state and is rebalancing data.
<li> "Migrates Outgoing" is the total number of partitions that must be sent to other nodes. As with the incoming number, a non-zero value means the cluster is in a dynamic state.
</ul>
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/6.png" alt="6" width="24" style="max-width: none"> 
		</td>
		<td>
			Namespaces: Information on each of the namespaces in the cluster.
		</td>
	</tr>
</table>

<img src="/docs/amc/assets/images/C_description_dashboard.png" alt="AMC dashboard" width="800">


