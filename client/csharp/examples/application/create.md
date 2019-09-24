---
title: Create a User and Tweet
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

These examples create and write User and Tweet records.

### Create User Record

#### Code

- Create a **new class**: `UserService.cs`.
- Add a reference to **Aerospike.Client**.
- Add a **private instance variable** of type `AerospikeClient`.
- Create a **public constructor** to accept one parameter of type `AerospikeClient`.

```cs
private AerospikeClient client;

public UserService(AerospikeClient c)
{
    this.client = c;
}
```
- Create a **new public** method: `createUser`.

```cs
public void createUser()
{
    Console.WriteLine("\n********** Create User **********\n");
    string username;
    string password;
    string gender;
    string region;
    string interests;

    // Get username
    Console.WriteLine("Enter username: ");
    username = Console.ReadLine();

    if (username != null && username.Length > 0)
    {
        // Get password
        Console.WriteLine("Enter password for " + username + ":");
        password = Console.ReadLine();

        // Get gender
        Console.WriteLine("Select gender (f or m) for " + username + ":");
        gender = Console.ReadLine().Substring(0, 1);

        // Get region
        Console.WriteLine("Select region (north, south, east or west) for " + username + ":");
        region = Console.ReadLine().Substring(0, 1);

        // Get interests
        Console.WriteLine("Enter comma-separated interests for " + username + ":");
        interests = Console.ReadLine();

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
        Bin bin7 = Bin.AsList("interests", interests.Split(',').ToList<object>());

        client.Put(wPolicy, key, bin1, bin2, bin3, bin4, bin5, bin6, bin7);

        Console.WriteLine("\nINFO: User record created!");
    }
} //createUser
```

### Creating a Tweet Record

#### Code

- Create a **new class**: `TweetService.cs`.
- Add a **reference** to _Aerospike.Client_.
- Add a **private instance variable** of type `AerospikeClient`.
- Create a **public constructor** to accept one parameter of type `AerospikeClient`.

```cs
private AerospikeClient client;

public TweetService(AerospikeClient c)
{
    this.client = c;
}
```

- Create a **new public method**: `createTweet`:

```cs
public void createTweet()
{
    Console.WriteLine("\n********** Create Tweet **********\n");
    Record userRecord = null;
    Key userKey = null;
    Key tweetKey = null;

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
            int nextTweetCount = int.Parse(userRecord.GetValue("tweetcount").ToString()) + 1;

            // Get tweet
            string tweet;
            Console.WriteLine("Enter tweet for " + username + ":");
            tweet = Console.ReadLine();

            // Write record
            WritePolicy wPolicy = new WritePolicy();
            wPolicy.recordExistsAction = RecordExistsAction.UPDATE;

            // Create timestamp to store along with the tweet so we can query, index and report on it
            long ts = getTimeStamp();

            tweetKey = new Key("test", "tweets", username + ":" + nextTweetCount);
            Bin bin1 = new Bin("tweet", tweet);
            Bin bin2 = new Bin("ts", ts);
            Bin bin3 = new Bin("username", username);

            client.Put(wPolicy, tweetKey, bin1, bin2, bin3);
            Console.WriteLine("\nINFO: Tweet record created!");

            // Update tweet count and last tweet'd timestamp in the user record
            client.Put(wPolicy, userKey, new Bin("tweetcount", nextTweetCount), new Bin("lasttweeted", ts));
        }
        else
        {
            Console.WriteLine("ERROR: User record not found!");
        }
    }
} //createTweet
```

- Open _Program.cs_ and uncomment these lines:
    
```cs
UserService us = new UserService(client);
TweetService ts = new TweetService(client);
us.createUser();
ts.createTweet();
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
- [Read the User Record](read.html)
