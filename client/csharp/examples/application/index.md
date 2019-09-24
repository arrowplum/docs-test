---
title: Application Tutorial
description: Design and implement of a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

This sample application demonstrates that Aerospike data structures on top of a key-value store are an effective way to write applications with Aerospike as the only database. To demonstrate, this sample describes the design and implementation of a Twitter-like application. 

This code is easy to follow and substantial enough to be a foundation for learning to leverage Aerospike's technology. It can also be used as a seed application that you can expand.

### Prerequisites 

- [Aerospike server](/docs/operations/install) installed and running.
- [Aerospike C# client](/docs/client/csharp/install) installed.

### Goals

Follow this tutorial to gain knowledge of:

- Key-value operations
- Secondary indexes
- Equality and Range filter queries
- UDFs (Record and Stream UDFs)
- Aggregations

### Application Features

This sample application demonstrates the use of:

- Writing user and tweet records
- Retrieving user and tweet records
- Updating a user record
- Querying the database
- Aggregating user statisics

{{#note}}
This sample application does not include features such as tracking `following` and `followers` and their tweets.
{{/note}}

To unserstand data structure and storage models, consider the data typically retrieved from a Twitter-like application:

- User records
- Tweets for a user
- Tweets for all users
- Users with a tweet count within a specified time period
- Users of a region with a tweet count within a specified time period

Map these to Aerospike data retrieval operations:

- Retrieve user records
    - **Get** operation
- Retrieve tweets for a user
    - **Batch** read operation *OR* a query using an **Equality** filter on `username`
- Retrieve tweets for all users
    - **Scan All** operation
- Retrieve users with `tweetcount` within a specified time period (Range)
    - **Secondary Index** on `tweetcount`
    - Query using **Range** filter on `tweetcount`
- Retrieve Users with `tweetcount` within the specified time period (Range), and aggregate users by region
    - **Secondary Index** on `tweetcount`
    - Query using **Range** filter on `tweetcount`
    - **Stream UDF** aggregates by user region

### Data Models

Data modeling is key for well performing applications. Data models not only define data structure and storage, but tend to drive the UI and UX of the application. This sample application uses the following data for modeling:

- User record
- The number of user tweets (`tweetcount`)


#### User Records

The user record bin stores the user profile. Note that:

- Only one record per user
- Key is `username`

Name | Description | Sample Data
--- | --- | ---
username | string | dash
password | string (plain text) | asrocks!
gender | string (Valid values = m or f) | m
region | string (Valid values = n (North), s (South), e (East), w (West) &mdash; Only stores the first letter) | w
lasttweeted* | Integer (Epoch timestamp of the last/most recent tweet) &mdash; Default = 0 | 1408574221
tweetcount** | Integer (Total user tweets) &mdash; Default = 0 | 32
interests | String array | photography,hiking,travel,house music

\* *lasttweeted* &mdash; Create a **secondary index**, and then run **time-based queries** (for example, top _n_ tweets within last _n_ mins).

\*\* *tweetcount* &mdash; Create a **secondary index**, and then **aggregate users** based on tweets (for example, users in a certain region who tweeted within a specified time period). 

#### Tweet

The _tweets_ bin stores all tweets for a user. Note that:

- Only one record per tweet
- Key is `username:counter`

Name | Description | Sample Data
--- | --- | ---
tweet | string | Put. A. Bird. On. It.
ts | Integer (Epoch timestamp of the tweet) | 1408574221
username* | string | dash

\* *username* &mdash; Create a **secondary index**, and then run queries (for example, retrieve all tweets for a user).

### Next Step
- [Application Setup](/docs/client/csharp/examples/application/setup.html)

