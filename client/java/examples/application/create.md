---
title: Create User and Tweet
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

This code creates and writes User and Tweet records.

### Creating a User Record

#### Code

- Create a **new class**: `UserService.java`.
- Add `import com.aerospike.client.AerospikeClient;`.
- Add a **private field** of type `AerospikeClient`.
- Create a **public constructor** to accept one parameter of type `AerospikeClient`:

```java
private AerospikeClient client;
private EclipseConsole console = new EclipseConsole();

public UserService(AerospikeClient c)
{
    this.client = c;
}
```

- Create a **new public method**: `createUser`:

```java
	public void createUser() throws AerospikeException {
		console.printf("\n********** Create User **********\n");

		String username;
		String password;
		String gender;
		String region;
		String interests;

		// Get username
		console.printf("Enter username: ");
		username = console.readLine();

		if (username != null && username.length() > 0) {
			// Get password
			console.printf("Enter password for " + username + ":");
			password = console.readLine();

			// Get gender
			console.printf("Select gender (f or m) for " + username + ":");
			gender = console.readLine().substring(0, 1);

			// Get region
			console.printf("Select region (north, south, east or west) for "
					+ username + ":");
			region = console.readLine().substring(0, 1);

			// Get interests
			console.printf("Enter comma-separated interests for " + username + ":");
			interests = console.readLine();

			// Write record
			WritePolicy wPolicy = new WritePolicy();
			wPolicy.recordExistsAction = RecordExistsAction.UPDATE;

			Key key = new Key("test", "users", username);
			Bin bin1 = new Bin("username", username);
			Bin bin2 = new Bin("password", password);
			Bin bin3 = new Bin("gender", gender);
			Bin bin4 = new Bin("region", region);
			Bin bin5 = new Bin("lasttweeted", 0);
			Bin bin6 = new Bin("tweetcount", 0);
			Bin bin7 = Bin.asList("interests", Arrays.asList(interests.split(",")));

			client.put(wPolicy, key, bin1, bin2, bin3, bin4, bin5, bin6, bin7);

			console.printf("\nINFO: User record created!");
		}
	} 
```

### Creating a Tweet Record

#### Code

- Create a **new class**: `TweetService.java`.
- Add `import com.aerospike.client.AerospikeClient;`.
- Add a **private field** of type `AerospikeClient`.
- Create a **public constructor** to accept one parameter of type `AerospikeClient`.

```java
	private AerospikeClient client;
	private EclipseConsole console = new EclipseConsole();

	public TweetService(AerospikeClient client) {
		this.client = client;
	}
```

- Create a **new public method**: `createTweet`:

```java

	public void createTweet() throws AerospikeException, InterruptedException {

		console.printf("\n********** Create Tweet **********\n");

		Record userRecord = null;
		Key userKey = null;
		Key tweetKey = null;

		// Get username
		String username;
		console.printf("\nEnter username:");
		username = console.readLine();

		if (username != null && username.length() > 0) {
			// Check if username exists
			userKey = new Key("test", "users", username);
			userRecord = client.get(null, userKey);
			if (userRecord != null) {
				int nextTweetCount = Integer.parseInt(userRecord.getValue(
						"tweetcount").toString()) + 1;

				// Get tweet
				String tweet;
				console.printf("Enter tweet for " + username + ":");
				tweet = console.readLine();

				// Write record
				WritePolicy wPolicy = new WritePolicy();
				wPolicy.recordExistsAction = RecordExistsAction.UPDATE;

				// Create timestamp to store along with the tweet so we can
				// query, index and report on it
				long ts = getTimeStamp();

				tweetKey = new Key("test", "tweets", username + ":"
						+ nextTweetCount);
				Bin bin1 = new Bin("tweet", tweet);
				Bin bin2 = new Bin("ts", ts);
				Bin bin3 = new Bin("username", username);

				client.put(wPolicy, tweetKey, bin1, bin2, bin3);
				console.printf("\nINFO: Tweet record created!\n");

				// Update tweet count and last tweet'd timestamp in the user
				// record
				updateUser(client, userKey, wPolicy, ts, nextTweetCount);
			} else {
				console.printf("ERROR: User record not found!\n");
			}
		}
	} //createTweet
```

- Open _Program.java_ and uncomment these lines:
    
```java
// UserService us = new UserService(client);
// TweetService ts = new TweetService(client);
// us.createUser();
// ts.createTweet();
```

### Running the Application

These messages and the application menu appear in the console:

```bash
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

Select **1** and press enter. The user setup screen appears:

```bash
Create User
Enter username:
```

After completion, a confirmation message appears:

```bash
INFO: User record created!
```

Follow the prompts to create a tweet for the user. After completion, a confirmation message appears:

```bash
INFO: Tweet record created!
```

### Next Step

- [Read the User Record](/docs/client/java/examples/application/read.html)
