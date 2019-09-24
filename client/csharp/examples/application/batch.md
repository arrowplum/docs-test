---
title: Batch Read Tweets
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

With our structured data stored in our data models, reading all tweets for a user is achieved either in a batch operation or in a secondary index (on `tweets.username`) query operation. These code examples initiate a batch operation to read tweet records.

First, we need a set of keys for all of the user’s tweet records. The key for a Tweet record is in format `username:<index>`. The `tweetcount` is stored as part of the user record. To generate all tweet record keys for a user, you need to iterate from 1 to `tweetcount` and concatenate `username` and `index`.

{{#note}}
This technique also works to retrieve all tweet records of users that this user is following, and the tweet records of the user’s followers.
{{/note}}

### Code

- Open _UserService.cs_.
- Create a **new public method**: `batchGetUserTweets`.

```cs
public void batchGetUserTweets()
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
            // Get how many tweets the user has
            int tweetCount = int.Parse(userRecord.GetValue("tweetcount").ToString());

            // Create an array of keys so we can initiate batch read operation
            Key[] keys = new Key[tweetCount];
            for (int i = 0; i < keys.Length; i++)
            {
                keys[i] = new Key("test", "tweets", (username + ":" + (i + 1)));
            }

            Console.WriteLine("\nHere's " + username + "'s tweet(s):\n");

            // Initiate batch read operation
            Record[] records = client.Get(null, keys);
            for (int j = 0; j < records.Length; j++)
            {
                Console.WriteLine(records[j].GetValue("tweet"));
            }
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
} //batchGetUserTweets
```
- Open _Program.cs_ and uncomment this line:
    
```cs
us.batchGetUserTweets();
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

Select **3** and press enter:

``` 
Enter username:
```

Enter your username. The user’s tweet records appear in the console.

### Next Step
- Write code to [Scan All Tweets](/docs/client/csharp/examples/application/scan.html)
