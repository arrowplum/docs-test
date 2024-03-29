---
title: Aggregate User Stats - Stream UDF
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

These examples use Stream UDFs to aggregate the number of users by region. Before aggregation, apply the Range filter to narrow the result set to users with `tweetcount` in the specified range. See [Stream UDFs](/docs/guide/stream_udf.html) for more information.

{{#note}}
You must have a basic knowledge of the [Lua programming language](http://www.lua.org/) to write UDFs.
{{/note}}

### Writing a Stream UDF

#### Code

- Create a **new file** _aggregationByRegion.lua_, and save it in the _udf_ folder.
- Open _aggregationByRegion.lua_ and add the following code:

```lua
local function aggregate_stats(map,rec)
  -- Examine value of 'region' bin in record rec and increment respective counter in the map
  if rec.region == 'n' then
      map['n'] = map['n'] + 1
  elseif rec.region == 's' then
      map['s'] = map['s'] + 1
  elseif rec.region == 'e' then
      map['e'] = map['e'] + 1
  elseif rec.region == 'w' then
      map['w'] = map['w'] + 1
  end
  -- return updated map
  return map
end

local function reduce_stats(a,b)
  -- Merge values from map b into a
  a.n = a.n + b.n
  a.s = a.s + b.s
  a.e = a.e + b.e
  a.w = a.w + b.w
  -- Return updated map a
  return a
end

function sum(stream)
return stream : aggregate(map{n=0,s=0,e=0,w=0},aggregate_stats) : reduce(reduce_stats)
end
```

This processes the incoming record stream, passes it to the aggregate UDF, and then to a reduce function. The `aggregate_stats` aggregate UDF accepts two parameters: 
- `map` contains four variables to store the number-of-users counter for the north, south, east, and west regions. The initial value is `0`. 
- `aggregate_stats` is called for each record as it flows in.

The reduced value of the map generated by `reduce_stats` reduce function returns. 

### Register and Execute Stream UDF

{{#note}}
UDF registration is included here to demonstrate syntax. We recommend using [aql](/docs/tools/aql) to register UDFs.
{{/note}}

#### Code

- Open _UserService.cs_.
- Create a **new public method**: `aggregateUsersByTweetCountByRegion`:

```cs
public void aggregateUsersByTweetCountByRegion()
{
    ResultSet rs = null;
    try
    {
        int min;
        int max;
        Console.WriteLine("\nEnter Min Tweet Count:");
        min = int.Parse(Console.ReadLine());
        Console.WriteLine("Enter Max Tweet Count:");
        max = int.Parse(Console.ReadLine());

        Console.WriteLine("\nAggregating users with " + min + "-" + max + " tweets by region. Hang on...\n");

        string luaDirectory = @"..\..\udf";
        LuaConfig.PackagePath = luaDirectory + @"\?.lua";

        string filename = "aggregationByRegion.lua";
        string path = Path.Combine(luaDirectory, filename);

        RegisterTask rt = client.Register(null, path, filename, Language.LUA);
        rt.Wait();

        string[] bins = { "tweetcount", "region" };
        Statement stmt = new Statement();
        stmt.SetNamespace("test");
        stmt.SetSetName("users");
        stmt.SetIndexName("tweetcount_index");
        stmt.SetBinNames(bins);
        stmt.SetFilters(Filter.Range("tweetcount", min, max));

        rs = client.QueryAggregate(null, stmt, "aggregationByRegion", "sum");

        if (rs.Next())
        {
            Dictionary<object, object> result = (Dictionary<object, object>)rs.Object;
            Console.WriteLine("Total Users in North: " + result["n"]);
            Console.WriteLine("Total Users in South: " + result["s"]);
            Console.WriteLine("Total Users in East: " + result["e"]);
            Console.WriteLine("Total Users in West: " + result["w"]);
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
} //aggregateUsersByTweetCountByRegion
```

- Open _Program.cs_ and uncomment this line:
    
```cs
// us.aggregateUsersByTweetCountByRegion();
```

### Running the Application

These messages and the application menu now appear in the console:

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

- Select **7** and press enter:

```bash 

Enter Min Tweet Count:
Enter Max Tweet Count:

```

- Enter the range and press enter. 
 The following appears on success:

```bash
Total Users in North: nTotal
Total Users in South: nTotal
Total Users in East: nTotal
Total Users in West: nTotal
```
