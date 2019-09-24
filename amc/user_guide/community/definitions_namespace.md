---
title: Definitions - Namespace
description: Learn how to view statistics and configuration variable values for different namespaces.
---

If you wish to view statistics and configuration variable values for different namespaces:

There are many variables, so you may need to look through the pages to find a specific one. For a list of complete variables, see the [Statistics and Variables page](/docs/operations/monitor/key_metrics).
<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24">
		</td>
		<td>
			Click on "Definitions" at the top.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24"> 
		</td>
		<td>
			Then select "Namespace".
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24"> 
		</td>
		<td>
			Select the namespace.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24"> 
		</td>
	         <td>
			The definition of secondary indexes. If the secondary index is not synced, queries using that index will not work.
		</td>
	</tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/5.png" alt="5" width="24">
                </td>
                <td>
                        You can create a new index or you can drop an existing index.
                </td>
        </tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/6.png" alt="6" width="24"> 
		</td>
		<td>
			Data on different sets. If you are using Enterprise Edition, it is possible to [synchronize multiple data centers on a set by set basis](/docs/operations/configure/cross-datacenter/replication). 
		</td>
	</tr>
        <tr>
		<td>
			<img src="/docs/amc/assets/images/7.png" alt="7" width="24"> 
		</td>
               	<td>
			The storage definition for the namespace. "Synced on all nodes" means that the system is stable and not rebalancing.
		</td>
        </tr>
</table>

<img src="/docs/amc/assets/images/c03_01_namespace_1.png" alt="namespace definitions" width="800">
