---
title: Scan All Tweets
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
 - /assets/styles/ui/steps.css
---

These examples reads all tweets for all users in the database using the `ScanAll` operation, and then displays them on the console.

### Code

- Open _TweetService.java_.
- Create a **new public method**: `scanAllTweetsForAllUsers`:

```java

	public void scanAllTweetsForAllUsers() {
		try {
			// Java Scan
			ScanPolicy policy = new ScanPolicy();
			policy.concurrentNodes = true;
			policy.priority = Priority.LOW;
			policy.includeBinData = true;

			client.scanAll(policy, "test", "tweets", new ScanCallback() {

				@Override
				public void scanCallback(Key key, Record record)
						throws AerospikeException {
					console.printf(record.getValue("tweet") + "\n");

				}
			}, "tweet");
		} catch (AerospikeException e) {
			System.out.println("EXCEPTION - Message: " + e.getMessage());
			System.out.println("EXCEPTION - StackTrace: "
					+ UtilityService.printStackTrace(e));
		}
	} //scanAllTweetsForAllUsers

```

- Open _Program.java_ and uncomment this line:
    
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
- [Update Password using Record UDF](/docs/client/java/examples/application/record_udf.html)
