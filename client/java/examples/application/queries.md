---
title: Query Users and Tweets
description: Design and implement a Twitter-like application written with Aerospike as the database.
styles:
  - /assets/styles/ui/steps.css
---

These code examples create a secondary index and run a query operation with an Equality filter to retrieve all tweet records for a user. Then the Range filter retrieves users based on their tweetcount (as defined in the min-max range).

### Retrieve All Tweets For A User

This example creates a secondary index on `tweets.username` to retrieve all tweet records and apply an **Equality** filter on `username`.

{{#note}}
Secondary index creation is included to demonstrate syntax. We recommend creating indexes using [aql](/docs/tools/aql).
{{/note}}

#### Code

- Open _TweetService.java_.
- Create a **new public method**: `queryTweetsByUsername`

```java
	public void queryTweetsByUsername() throws AerospikeException {
		
		console.printf("\n********** Query Tweets By Username **********\n");
		
		RecordSet rs = null;
		try {
			IndexTask task = client.createIndex(null, "test", "tweets",
					"username_index", "username", IndexType.STRING);
			task.waitTillComplete(100);

			// Get username
			String username;
			console.printf("\nEnter username:");
			username = console.readLine();

			if (username != null && username.length() > 0) {
				String[] bins = { "tweet" };
				Statement stmt = new Statement();
				stmt.setNamespace("test");
				stmt.setSetName("tweets");
				stmt.setIndexName("username_index");
				stmt.setBinNames(bins);
				stmt.setFilters(Filter.equal("username", username));

				console.printf("\nHere's " + username + "'s tweet(s):\n");

				rs = client.query(null, stmt);
				while (rs.next()) {
					Record r = rs.getRecord();
					console.printf(r.getValue("tweet").toString() + "\n");
				}
			} else {
				console.printf("ERROR: User record not found!\n");
			}
		} finally {
			if (rs != null) {
				// Close record set
				rs.close();
			}
		}
	} //queryTweetsByUsername

```

### Retrieve All User Records using `tweetcount`

To retrieve all user records using `tweetcount` (as defined in the min-max range), create a secondary index for `users.tweetcount` and apply a  **Range** filter on `tweetcount`.

{{#note}}
Secondary index creation is included to demonstrate syntax. We recommend creating indexes using [aql](/docs/tools/aql).
{{/note}}

#### Code

- Open _TweetService.java_.
- Create a **new public method**: `queryUsersByTweetCount`

```java
	public void queryUsersByTweetCount() throws AerospikeException {

		console.printf("\n********** Query Users By Tweet Count Range **********\n");

		RecordSet rs = null;
		try {
			IndexTask task = client.createIndex(null, "test", "users",
					"tweetcount_index", "tweetcount", IndexType.NUMERIC);
			task.waitTillComplete(100);

			// Get min and max tweet counts
			int min;
			int max;
			console.printf("\nEnter Min Tweet Count:");
			min = Integer.parseInt(console.readLine());
			console.printf("Enter Max Tweet Count:");
			max = Integer.parseInt(console.readLine());

			console.printf("\nList of users with " + min + "-" + max
					+ " tweets:\n");

			String[] bins = { "username", "tweetcount", "gender" };
			Statement stmt = new Statement();
			stmt.setNamespace("test");
			stmt.setSetName("users");
			stmt.setBinNames(bins);
			stmt.setFilters(Filter.range("tweetcount", min, max));

			rs = client.query(null, stmt);
			while (rs.next()) {
				Record r = rs.getRecord();
				console.printf(r.getValue("username") + " has "
						+ r.getValue("tweetcount") + " tweets\n");
			}
		} finally {
			if (rs != null) {
				// Close record set
				rs.close();
			}
		}
	} //queryUsersByTweetCount
```

- Open _Program.java_ and uncomment these lines:
    
```java

// ts.queryTweetsByUsername();
// ts.queryUsersByTweetCount();
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

Select **6** and press enter:

```
Enter username:
```

Enter your username. The user’s tweets appear in the console.

The client then displays:

```
Enter Min Tweet Count:
Enter Max Tweet Count:
```

Enter the range and press enter. A list of Users and their tweetcount appears in the console.

### Next Step

- [Aggregate Users](/docs/client/java/examples/application/aggregate.html)
