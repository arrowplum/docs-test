---
title: Application Setup
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

This example sets up the application environment and writes 'skeleton' console application code to:

- Connect to your Aerospike Server
- Present a menu to perform application features
- Disconnect from your Aerospike Server

{{#note}}
These examples apply application features one at a time. However, you can download the entire solution from [GitHub](https://github.com/aerospike/aerospike-sample-applications).
{{/note}}

### Environment

1. Open **Visual Studio**
1. Select **File > New > Project** and follow the prompts to create a new Console application. 
1. Select **Project > Properties**.
1. Select **Build** in the left pane. 
1. Select **x64** as the **Platform target**.
1. Select **File > Save Selected Items**.
1. Select **Project > Add Reference**.
1. Click **Browse**.

	Browse to local folder /AerospikeClient/bin/x64/Release where Aerospike dlls were created during the Build process during Setup.
1. Select **AerospikeClient.dll**.
1. Click **OK**.
1. Select **Build > Rebuild Solution**.

{{#note}}
Resolve all errors before proceeding.
{{/note}}

### Code

- In _Program.cs_, add a reference to **Aerospike.Client**
- In the `Main` method, create an instance of `AerospikeClient` and two other variables to store the Aerospike server IP and port.
- Create a **try catch block** to catch the appropriate exceptions and **close AerospikeClient connection** in the **finally block**. 
	Closing the client connection is critical to release resources such as sockets and threads.

```cs
AerospikeClient client = null;
string asServerIP = "127.0.0.1";
int asServerPort = 3000;

try
{
    Console.WriteLine("INFO: Connecting to Aerospike cluster...");
    // Establish connection
    client = new AerospikeClient(asServerIP, asServerPort);
    // Check to see if the cluster connection succeeded
    if (client.Connected)
    {
        Console.WriteLine("INFO: Connection to Aerospike cluster succeeded!\n");        

        // Create instance of UserService
        // UserService us = new UserService(client);
        // Create instance of TweetService
        // TweetService ts = new TweetService(client);

        // Present options
        Console.WriteLine("What would you like to do:");
        Console.WriteLine("1> Create User And Tweet");
        Console.WriteLine("2> Read User Record");
        Console.WriteLine("3> Batch Read Tweets For User");
        Console.WriteLine("4> Scan All Tweets For All Users");
        Console.WriteLine("5> Record UDF -- Update User Password");
        Console.WriteLine("6> Query Tweets By Username And Users By Tweet Count Range");
        Console.WriteLine("7> Stream UDF -- Aggregation Based on Tweet Count By Region");
        Console.WriteLine("0> Exit");
        Console.Write("\nSelect 0-7 and hit enter:");
        byte feature = byte.Parse(Console.ReadLine());

        if (feature != 0)
        {
            switch (feature)
            {
                case 1:
                    Console.WriteLine("\n********** Your Selection: Create User And Tweet **********\n");
                    // us.createUser();
                    // ts.createTweet();
                    break;
                case 2:
                    Console.WriteLine("\n********** Your Selection: Read User Record **********\n");
                    // us.readUser();
                    break;
                case 3:
                    Console.WriteLine("\n********** Your Selection: Batch Read Tweets For User **********\n");
                    // us.batchGetUserTweets();
                    break;
                case 4:
                    Console.WriteLine("\n**********  Your Selection: Scan All Tweets For All Users **********\n");
                    // ts.scanAllTweetsForAllUsers();
                    break;
                case 5:
                    Console.WriteLine("\n********** Your Selection: Record UDF -- Update User Password **********\n");
                    // us.updatePasswordUsingUDF();
                    break;
                case 6:
                    Console.WriteLine("\n**********  Your Selection: Query Tweets By Username And Users By Tweet Count Range **********\n");
                    // ts.queryTweetsByUsername();
                    // ts.queryUsersByTweetCount();
                    break;
                case 7:
                    Console.WriteLine("\n**********  Your Selection: Stream UDF -- Aggregation Based on Tweet Count By Region **********\n");
                    // us.aggregateUsersByTweetCountByRegion();
                    break;
                default:
                    Console.WriteLine("\n********** Invalid Selection **********\n");
                    break;
            }
        }
    }
    else
    {
        Console.Write("ERROR: Connection to Aerospike cluster failed! Please check IP & Port settings and try again!");
    }
}
catch (AerospikeException e)
{
    Console.WriteLine("AerospikeException - Message: " + e.Message);
    Console.WriteLine("AerospikeException - StackTrace: " + e.StackTrace);
}
catch (Exception e)
{
    Console.WriteLine("Exception - Message: " + e.Message);
    Console.WriteLine("Exception - StackTrace: " + e.StackTrace);
}
finally
{
    if (client != null && client.Connected)
    {
        // Close Aerospike server connection
        client.Close();
    }
    Console.ReadLine();
}
```

### Running the Application

These messages appear in the console and the menu:

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

Select 0-7 and hit enter:
```

{{#note}}
If “Connection to Aerospike cluster failed!” appears, ensure that the Aerospike Server is running and available, and confirm that `asServerIP` and `asServerPort` are properly set.
{{/note}}

### Next Step
- [Create a User and Tweet](/docs/client/csharp/examples/application/create.html) 

