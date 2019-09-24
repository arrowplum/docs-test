---
title: User Management
description: Learn how to use user management feature in AMC.
---
This feature is available for security enabled Aerospike clusters only. User with "user-admin" roles can access this feature in AMC. With the help of this, you can create users,change access,password of users, drop users. 

<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24" style="max-width: none">
		</td>
		<td>
			Click on manage tab
		</td>
	</tr>
		<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24" style="max-width: none">
		</td>
		<td>
		   	Click on User Management tab. 
		</td>
	</tr>
		<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24" style="max-width: none">
		</td>
		<td>
			In User management table you can see users and their roles.
		</td>
	</tr>
		<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24" style="max-width: none">
		</td>
		<td>
			You can edit users by clicking on the edit icon.
		</td>
	</tr>
		<tr>
		<td>
			<img src="/docs/amc/assets/images/5.png" alt="5" width="24" style="max-width: none">
		</td>
		<td>
			By default 10 users will be displayed. You can change number of users per page or paginate through all the users.
		</td>
	</tr>
   	<tr>
		<td>
			<img src="/docs/amc/assets/images/6.png" alt="6" width="24" style="max-width: none">
		</td>
		<td>
			You can add new user with the help of Add new user button.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/7.png" alt="7" width="24" style="max-width: none">
		</td>
		<td>
			Manage tab functionalities accessible to users according to roles, In this diagram user have no access to back up and restore functionalities that's why those tabs shown in red colour and click on those user will get error.
	        </td>
	</tr>
	</table>
<img src="/docs/amc/assets/images/E_user_management.png" alt="Alerts" width="800">
<br/>

Clicking on Add new user, "Add user" popup will be displayed. You can create users by assigning a user name, password and roles. 
Note: User name (unique), Password and at least one role is mandatory to create new user.

<img src="/docs/amc/assets/images/E_add_user.png" alt="Alerts" width="800">
