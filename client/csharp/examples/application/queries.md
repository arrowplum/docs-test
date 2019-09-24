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

- Open _TweetService.cs_.
- Create a **new public method**: `queryTweetsByUsername`.

```cs
public void queryTweetsByUsername()
{
    Console.WriteLine("\n********** Query Tweets By Username **********\n");

    RecordSet rs = null;
    try
    {
   
        IndexTask task = client.CreateIndex(null, "test", "tweets", "username_index", "username", IndexType.STRING);
        task.Wait();

        // Get username
        string username;
        Console.WriteLine("\nEnter username:");
        username = Console.ReadLine();

        if (username != null && username.Length > 0)
        {
            string[] bins = { "tweet" };
            Statement stmt = new Statement();
            stmt.SetNamespace("test");
            stmt.SetSetName("tweets");
            stmt.SetIndexName("username_index");
            stmt.SetBinNames(bins);
            stmt.SetFilters(Filter.Equal("username", username));

            Console.WriteLine("\nHere's " + username + "'s tweet(s):\n");

            rs = client.Query(null, stmt);
            while (rs.Next())
            {
                Record r = rs.Record;
                Console.WriteLine(r.GetValue("tweet"));
            }
        }
        else
        {
            Console.WriteLine("ERROR: User record not found!");
        }
    }
    finally
    {
        if (rs != null)
        {
            // Close record set
            rs.Close();
        }
    }
} //queryTweetsByUsername
```

### Retrieve All Users with Tweetcount Between min-max Range

To retrieve all user records using `tweetcount` (as defined in the min-max range), create a secondary index for `users.tweetcount` and apply a  **Range** filter on `tweetcount`.

{{#note}}
Secondary index creation is included to demonstrate syntax. We recommend creating indexes using [aql](/docs/tools/aql).
{{/note}}

#### Code
 
- Open _TweetService.cs_.
- Create a **new public method**: `queryUsersByTweetCount`.

```cs
public void queryUsersByTweetCount()
{
    Console.WriteLine("\n********** Query Users By Tweet Count Range **********\n");

    RecordSet rs = null;
    try
    {

        IndexTask task = client.CreateIndex(null, "test", "users", "tweetcount_index", "tweetcount", IndexType.NUMERIC);
        task.Wait();

        // Get min and max tweet counts
        int min;
        int max;
        Console.WriteLine("\nEnter Min Tweet Count:");
        min = int.Parse(Console.ReadLine());
        Console.WriteLine("Enter Max Tweet Count:");
        max = int.Parse(Console.ReadLine());

        string[] bins = { "username", "tweetcount" };
        Statement stmt = new Statement();
        stmt.SetNamespace("test");
        stmt.SetSetName("users");
        stmt.SetIndexName("tweetcount_index");
        stmt.SetBinNames(bins);
        stmt.SetFilters(Filter.Range("tweetcount", min, max));

        Console.WriteLine("\nList of users with " + min + "-" + max + " tweets:\n");

        rs = client.Query(null, stmt);
        while (rs.Next())
        {
            Record r = rs.Record;
            Console.WriteLine(r.GetValue("username") + " has " + r.GetValue("tweetcount") + " tweets");
        }
    }
    finally
    {
        if (rs != null)
        {
            // Close record set
            rs.Close();
        }
    }
} //queryUsersByTweetCount
```

- Open _Program.cs_ and uncomment these lines:
    
```cs
ts.queryTweetsByUsername();
ts.queryUsersByTweetCount();
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

- Select **6** and press enter:

```
Enter username:
```

- Enter your username. 
 The user’s tweets appear in the console.

 The client then displays:

```
Enter Min Tweet Count:
Enter Max Tweet Count:
```

- Enter the range and press enter. A list of Users and their tweetcount appears in the console.

### Next Step

- [Aggregate Users](/docs/client/csharp/examples/application/aggregate.html)
