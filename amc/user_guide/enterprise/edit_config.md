---
title: Edit Configuration
description: Learn how to change configuration variables from the management console.
---

{{#todo markdown=true}}
- Need link to config variable page.
{{/todo}}

From the Admin Console, you can make dynamic changes to most of the configuration variables. You can do this while the cluster is still up. However, note that these changes are not made to the Aerospike configuration file. Restarts of the node will result in a return to the values specified in the configuration file.

For a description of the meaning of the variables, please see the [configuration variables page](/docs/reference/configuration)

<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24">
		</td>
		<td>
			Click on the "Manage" at the top.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24"> 
		</td>
		<td>
			Choose whether you would like to change the vairables for nodes, namespace, or XDR.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24"> 
		</td>
		<td>
			Make changes to the appropriate configuration variables.
		</td>
	</tr>
</table>

<img src="/docs/amc/assets/images/E_config_editor.png" alt="Edit Config" width="800">


