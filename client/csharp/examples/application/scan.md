---
title: Scan All Tweets
description:  Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

This code reads all tweets for all users in the database using the `ScanAll` operation, and then displays them on the console.

### Code

- Open _TweetService.cs_.
- Create a **new public method**: `scanAllTweetsForAllUsers`:

```cs
public void scanAllTweetsForAllUsers()
{
    ScanPolicy policy = new ScanPolicy();
    policy.includeBinData = true;
    client.ScanAll(policy, "test", "tweets", scanTweetsCallback, "tweet");
} //scanAllTweetsForAllUsers
```

- Create a **new public method**: `scanTweetsCallback`:

```cs
public void scanTweetsCallback()
{
    Console.WriteLine(record.GetValue("tweet"));
} //scanTweetsCallback
```

- Open _Program.cs_ and uncomment this line:
    
```cs
ts.scanAllTweetsForAllUsers();
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

Select **4** and press enter. **All** user tweets display on the console.

### Next Step
- [Update Password using Record UDF](/docs/client/csharp/examples/application/record_udf.html)
