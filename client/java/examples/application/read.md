---
title: Read User Record
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

This example reads the User record and outputs to the console.

### Code

- Open _UserService.java_.
- Create a **new public method**: `getUser`

```java

	public void getUser() throws AerospikeException {
		Record userRecord = null;
		Key userKey = null;

		// Get username
		String username;
		console.printf("\nEnter username:");
		username = console.readLine();

		if (username != null && username.length() > 0) {
			// Check if username exists
			userKey = new Key("test", "users", username);
			userRecord = client.get(null, userKey);
			if (userRecord != null) {
				console.printf("\nINFO: User record read successfully! Here are the details:\n");
				console.printf("username:   " + userRecord.getValue("username")
						+ "\n");
				console.printf("password:   " + userRecord.getValue("password")
						+ "\n");
				console.printf("gender:     " + userRecord.getValue("gender") + "\n");
				console.printf("region:     " + userRecord.getValue("region") + "\n");
				console.printf("tweetcount: " + userRecord.getValue("tweetcount") + "\n");
				console.printf("interests:  " + userRecord.getValue("interests") + "\n");
			} else {
				console.printf("ERROR: User record not found!\n");
			}		
		} else {
			console.printf("ERROR: User record not found!\n");
		}		
	} //getUser
```

- Open _Program.java_ and uncomment this line:
    
```java
// us.getUser();
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
- [Batch Read Tweets](/docs/client/java/examples/application/batch.html)
