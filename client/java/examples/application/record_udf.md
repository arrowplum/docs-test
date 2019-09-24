---
title: Update Password - Record UDF
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

These code examples use Record user-defined functions (UDFs) to update a user password. See [Record UDFs](/docs/guide/record_udf.html) for more information.

{{#note}}
You must have a basic knowledge of the [Lua Programming Language](http://www.lua.org/) to write UDFs.
{{/note}}

### Write a Record UDF

#### Code

1. Create a **new folder** _udf_ at the application root. 
1. Create a **new file** _updateUserPwd.lua_ and save it to _udf_.
1. Open _updateUserPwd.lua_ and add the following code:

```lua
function updatePassword(topRec,pwd)
   -- Assign new password to the user record
   topRec['password'] = pwd
   -- Update user record
   aerospike:update(topRec)
   -- return new password
   return topRec['password']
end
```

### Register and Execute a Record UDF

{{#note}}
UDF registration is included to demonstrate syntax. We recommend using aql to register UDFs.
{{/note}}

#### Code

- Open _UserService.java_.
- Create a **new public method**: `updatePasswordUsingUDF`

```java

	public void updatePasswordUsingUDF() throws AerospikeException
	{
		Record userRecord = null;
		Key userKey = null;

		// Get username
		String username;
		console.printf("\nEnter username:");
		username = console.readLine();

		if (username != null && username.length() > 0)
		{
			// Check if username exists
			userKey = new Key("test", "users", username);
			userRecord = client.get(null, userKey);
			if (userRecord != null)
			{
				// Get new password
				String password;
				console.printf("Enter new password for " + username + ":");
				password = console.readLine();
				LuaConfig.SourceDirectory = "udf";
				File udfFile = new File("udf/updateUserPwd.lua");

				RegisterTask rt = client.register(null, udfFile.getPath(),
						udfFile.getName(), Language.LUA);
				rt.waitTillComplete(100);

				String updatedPassword = client.execute(null, userKey, "updateUserPwd", "updatePassword", Value.get(password)).toString();
				console.printf("\nINFO: The password has been set to: " + updatedPassword);
			}
			else
			{
				console.printf("ERROR: User record not found!");
			}
		}
		else
		{
			console.printf("ERROR: User record not found!");
		}
	} //updatePasswordUsingUDF
```

- Open _Program.java_ and uncomment this line:
    
```java
// us.updatePasswordUsingUDF();
```

### Running the Application

These messages and the application menu now appear in the console:

```
INFO: Connecting to Aerospike cluster...
INFO: Connection to Aerospike cluster succeeded!

What would you like to do:
1> Create A User And A Tweet
2> Read A User Record
3> Batch Read Tweets For A User
4> Scan All Tweets For All Users
5> Record UDF -- Update User Password
6> Query Tweets By Username And Users By Tweet Count Range
7> Stream UDF -- Aggregation Based on Tweet Count By Region
0> Exit
```
Select **5** and press enter:

```
Enter username:
```

Enter your username:

```
Enter new password for <username>:
```

Enter the new password and press enter:

```
INFO: The password has been set to: <new password>
```

### Next Step
- [Query Users and Tweets](/docs/client/java/examples/application/queries.html)
