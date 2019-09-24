---
title: Latency Charts
description: Learn how you can view latency values in the Enterprise Edition management console.
---
One important function of the AMC is to show latencies as measured on the server. By comparing the values on the server with those on the client, you can determine if any latency issues are due to the server, the network, or the client. 


<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24">
		</td>
		<td>
			Click on the "Latency" tab at the top.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24"> 
		</td>
		<td>
			The top chart will show the current latency in different colored graphs. 
			<ul>
				<li>Green represents the number of transactions that took less than 1 millisecond.</li>
				<li>Yellow represents the number of transactions that took between 1-8 milliseconds.</li>
				<li>Orange represents the number of transactions that took between 8-64 milliseconds.</li>
				<li>Red represents the number of transactions that took longer than 64 milliseconds.</li>
			</ul>
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24"> 
		</td>
		<td>
			Below you can select the graph for the node and transaction type.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24"> 
		</td>
		<td>
			You can change the time interval for the graphs.
		</td>
	</tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/5.png" alt="5" width="24">
                </td>
                <td>
                        On hover you can see the data in the graphs at that particular point.
                </td>
        </tr>

</table>

<img src="/docs/amc/assets/images/e05_latency.png" alt="Latency Chart" width="800">


