---
title: Role Management 
description: Learn how to use the Role_Management feature in AMC.
---
This feature is available in Aerospike when security is enabled. With the help of this feature you can create, delete or update role. 
Role is set of privileges. Privileges consist of permissions and scopes. There are 6 default permissions available- read, read-write,read-write-udf,sys-admin,user-admin, data-admin. Scopes defines where permissions applied globally, per namespace, or per namespace and set.
Note: User admin, sys-admin and data-admin permissions can only have global scopes. 
By default 6 roles are available – read, read-write, read-write-udf, sys-admin and user-admin, data-admi. You can create multiple roles by assigning group of privileges. 
<table border="0">
        <tr>
                <td>
                        <b>Roles</b>
                </td>
                <td>
                        <b>Privilleges</b>
                </td>

        </tr>
        <tr>
                <td>
                        "data-admin"
                </td>
                <td>
                        "data-admin" 
                </td>
        </tr>
        <tr>
                <td>
                        "read" 
                </td>
                <td>
                        "read"
                </td>
        </tr>
        <tr>
                <td>
                        "read-write"
                </td>
                <td>
                        "read-write"
                </td>
        </tr>
        <tr>
                <td>
                        "read-write-udf"
                </td>
                <td>
                        "read-write-udf"
                </td>
        </tr>
        <tr>
                <td>
                        "superuser"
                </td>
                <td>
                        "user-admin, sys-admin, read-write-udf"
                </td>
        </tr>
        <tr>
                <td>
                        "sys-admin"
                </td>
                <td>
                        "sys-admin"
                </td>
        </tr>
        <tr>
                <td>
                        "user-admin"
                </td>
                <td>
                        "user-admin" 
                </td>
        </tr>
</table>

Please refer steps to create, delete and update role through AMC.

<table border="0">
	<tr>
		<td>
			<img src="/docs/amc/assets/images/1.png" alt="1" width="24">
		</td>
		<td>
			Click on "Manage" tab at the top.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/2.png" alt="2" width="24"> 
		</td>
		<td>
			Click on Role Management on the left panel.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/3.png" alt="3" width="24"> 
		</td>
		<td>
			In Role Management you can see roles and privilleges.
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/4.png" alt="4" width="24"> 
		</td>
		<td>
			You can edit role by click on the edit icon
		</td>
	</tr>
	<tr>
		<td>
			<img src="/docs/amc/assets/images/5.png" alt="5" width="24"> 
		</td>
		<td>
			By default 10 roles will be displayed per page. You can change number of roles per page or paginate through all the roles.
		</td>
	</tr>
		<tr>
		<td>
			<img src="/docs/amc/assets/images/6.png" alt="6" width="24"> 
		</td>
		<td>
			You can add new role with the help of Add new Role button.
		</td>
	</tr>
</table>

<img src="/docs/amc/assets/images/E_role_management.png" alt="role_management" width="800">
<br/>

Clicking on Add new Role, "Add Role" pop-up will be displayed. You can create role by assigning a role name, privileges, and scopes. Note: All fields are mandatory and role name required to be unique.

<img src="/docs/amc/assets/images/E_add_role.png" alt="role_management" width="800">
<br/>

For more details please refer this link:
https://www.aerospike.com/docs/guide/security.html
