---
title: Read User Record
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

This example reads the User record and outputs to the console.

### Code

- Open _UserService.cs_.
- Create a **new public method**: `readUser`.

```cs
public void readUser()
{
    Record userRecord = null;
    Key userKey = null;

    // Get username
    string username;
    Console.WriteLine("\nEnter username:");
    username = Console.ReadLine();

    if (username != null && username.Length > 0)
    {
        // Check if User record exists
        userKey = new Key("test", "users", username);
        userRecord = client.Get(null, userKey);
        if (userRecord != null)
        {
            Console.WriteLine("\nINFO: User record read successfully! Here are the details:\n");
            Console.WriteLine("username:     " + userRecord.GetValue("username"));
            Console.WriteLine("password:     " + userRecord.GetValue("password"));
            Console.WriteLine("gender:       " + userRecord.GetValue("gender"));
            Console.WriteLine("region:       " + userRecord.GetValue("region"));
            Console.WriteLine("tweetcount:   " + userRecord.GetValue("tweetcount"));
            List<object> interests = (List<object>) userRecord.GetValue("interests");
            Console.WriteLine("interests:    " + interests.Aggregate((x, y) => x + "," + y));
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
} //readUser
```

- Open _Program.cs_ and uncomment this line:
    
```cs
us.readUser();
```

### Running the Application

These messages and the application menu appear in the console:

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

Select **2** and press enter:

```
Enter username:
```

Enter your username. The user details now appear in the console.

### Next Step
- [Batch Read Tweets](batch.html)
