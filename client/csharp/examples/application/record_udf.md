---
title: Update Password - Record UDF
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

These code examples use Record user-defined functions (UDFs) to update a user password. See [Record UDFs](/docs/guide/record_udf.html) for more information.


### Write Record UDF

#### Code

- Create a **new folder** _udf_ at the application root. 
- Create a **new file** _updateUserPwd.lua_ and save it to _udf_.
- Open _updateUserPwd.lua_ and add the following code:

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

### Register and Execute Record UDF

{{#note}}
UDF registration is included to demonstrate syntax. We recommend using aql to register UDFs.
{{/note}}

#### Code

- Open _UserService.cs_.
- Create a **new public method**: `updatePasswordUsingUDF`:

```cs
public void updatePasswordUsingUDF()
{
    Record userRecord = null;
    Key userKey = null;

    // Get username
    string username;
    Console.WriteLine("\nEnter username:");
    username = Console.ReadLine();

    if (username != null && username.Length > 0)
    {
        // Check if username exists
        userKey = new Key("test", "users", username);
        userRecord = client.Get(null, userKey);
        if (userRecord != null)
        {
            // Get new password
            string password;
            Console.WriteLine("Enter new password for " + username + ":");
            password = Console.ReadLine();

            string luaDirectory = @"..\..\udf";
            LuaConfig.PackagePath = luaDirectory + @"\?.lua";
            string filename = "updateUserPwd.lua";
            string path = Path.Combine(luaDirectory, filename);
            RegisterTask rt = client.Register(null, path, filename, Language.LUA);
            rt.Wait();

            string updatedPassword = client.Execute(null, userKey, "updateUserPwd", "updatePassword", Value.Get(password)).ToString();
            Console.WriteLine("\nINFO: The password has been set to: " + updatedPassword);
        }
        else
        {
            Console.WriteLine("ERROR: User record not found!");
        }
    }
    else
    {
        Console.WriteLine("ERROR: User record not found!");
    }
} //updatePasswordUsingUDF
```

- Open _Program.cs_ and uncomment this line:
    
```cs
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
- Select **5** and press enter:

```
Enter username:
```

- Enter your username:

```
Enter new password for <username>:
```

- Enter the new password and press enter:

```
INFO: The password has been set to: <new password>
```

### Next Step
- [Query Users and Tweets](/docs/client/csharp/examples/application/queries.html)
