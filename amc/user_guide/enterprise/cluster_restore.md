---
title: Cluster Restore
description: Learn how to restore from a backup.
---

{{#todo markdown=true}}
- Need link to CLI backup command.
{{/todo}}

As important as backup a database is the ability to restore the backup. Aerospike allows you to do restores while the cluster is still operating.

You can also use the [command line version of the restore program](/docs/tools/backup/asrestore.html).

{{#note}}
For Aerospike with security enabled the user needs to have sys-admin and read-write roles in order to access this functionality.
{{/note}}

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
			Click on "Cluster Restore" on the left.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24"> 
		</td>
		<td>
			Enter the information for the restoration. The username and password for the server that will be running the backup.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24"> 
		</td>
		<td>
			As an advanced option, you may opt to override the namespace of the data.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/5.png" alt="5" width="24"> 
		</td>
		<td>
			The number of threads will control how fast the restoration will run. Be careful not to run it too high or it will interfere with the database operations.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/6.png" alt="6" width="24"> 
		</td>
		<td>
			If you check the "Ignore Generation Number" checkbox, all data will be overwritten regardless as to the generation number.
		</td>
	</tr>
</table>

<img src="/docs/amc/assets/images/E_cluster_restore.png" alt="cluster restore" width="800">


