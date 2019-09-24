---
title: Cluster Backup
description: Learn how to perform a backup while the cluster is operating.
---

{{#todo markdown=true}}
- Need link to CLI backup command. Can't seem to find it.
{{/todo}}

One important function for any database is backing it up. Aerospike allows you to do backups while the cluster is still operating. This is done one namespace at a time. The output will be spread throughout a set of text files in a directory of the admin's choosing.

You can also use the [command line version of the backup program](/docs/tools/backup/asbackup.html).

{{#note}}
For Aerospike with security enabled the user needs to have sys-admin and read roles in order to access this functionality.
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
			Click on "Cluster Backup" on the left.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24"> 
		</td>
		<td>
			Enter the information for the backup. The username and password for the server that will be running the backup.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24"> 
		</td>
		<td>
			As an advanced option, you may opt to backup only a specific set of data.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/5.png" alt="5" width="24"> 
		</td>
		<td>
			The backup priority can be set to higher or lower in order to minimize impact to the database.
		</td>
	</tr>
</table>

<img src="/docs/amc/assets/images/E_cluster_backup.png" alt="cluster backup" width="800">


