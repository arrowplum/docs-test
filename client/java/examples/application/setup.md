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

This example uses [Maven](http://maven.apache.org/) as the configuration and build environment. All dependencies and the build are described in _pom.xml_.

In the dependencies section, include `aerospike-client`:

```xml

<dependencies>
	<dependency>
		<groupId>com.aerospike</groupId>
		<artifactId>aerospike-client</artifactId>
		<version>3.0.30</version>
	</dependency>
</dependencies>
```

The Aerospike Java client automatically downloads and installs in your local Maven repository.

The `maven-assembly-plugin` builds a runnable jar in the _target_ directory:

```xml

<plugin>
	<artifactId>maven-assembly-plugin</artifactId>
	<configuration>
		<descriptorRefs>
			<descriptorRef>jar-with-dependencies</descriptorRef>
		</descriptorRefs>
		<archive>
			<manifest>
				<addClasspath>true</addClasspath>
				<mainClass>com.aerospike.examples.tweetaspike.Program</mainClass>
			</manifest>
		</archive>
		<finalName>${project.artifactId}-${project.version}-full</finalName>
		<appendAssemblyId>false</appendAssemblyId>
	</configuration>
	<executions>
		<execution>
			<id>make-my-jar-with-dependencies</id>
			<phase>package</phase>
			<goals>
				<goal>single</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

Build:

```bash
mvn clean package
```

The jar file now resides in the _target_ directory:

`tweetaspike-example-application-1.0.0-full.jar`

{{#note}}
Resolve all errors before proceeding.
{{/note}}

### Code

- In _com.aerospike.examples.tweetaspike.Program.java_, add a reference to `AerospikeClient`.
- In the constructor method, create an instance of `AerospikeClient` using the server IP and port.
- Create a `finalize()` method to close the client connection to the cluster. Closing client connection is critical for freeing resources such as sockets and threads.
- Create a `work()` method and call it from your main.

**Constructor Example**

```java
ClientPolicy cPolicy = new ClientPolicy();
cPolicy.timeout = 500;
this.client = new AerospikeClient(cPolicy, this.seedHost, this.port);
```
`finalize()` method:

```java
protected void finalize() throws Throwable {
	if (this.client != null){
		this.client.close();
	}
};
```

`work()` method:

```java

try {
	console.printf("INFO: Connecting to Aerospike cluster...");

	// Establish connection to Aerospike server

	if (client == null || !client.isConnected()) {
		console.printf("\nERROR: Connection to Aerospike cluster failed! Please check the server settings and try again!");
		console.readLine();
	} else {
		console.printf("\nINFO: Connection to Aerospike cluster succeeded!\n");

		// Create instance of UserService
		// UserService us = new UserService(client);
		// Create instance of TweetService
		// TweetService ts = new TweetService(client);
		// Create instance of UtilityService
		// UtilityService util = new UtilityService(client);

		// Present options
		console.printf("\nWhat would you like to do:\n");
		console.printf("1> Create A User And A Tweet\n");
		console.printf("2> Read A User Record\n");
		console.printf("3> Batch Read Tweets For A User\n");
		console.printf("4> Scan All Tweets For All Users\n");
		console.printf("5> Record UDF -- Update User Password\n");
		console.printf("6> Query Tweets By Username And Users By Tweet Count Range\n");
		console.printf("7> Stream UDF -- Aggregation Based on Tweet Count By Region\n");
		console.printf("0> Exit\n");
		console.printf("\nSelect 0-7 and hit enter:\n");
		int feature = Integer.parseInt(console.readLine());
		
		if (feature != 0) {
			switch (feature) {
			case 1:
				console.printf("\n********** Your Selection: Create User And A Tweet **********\n");
				// us.createUser();
				// ts.createTweet();
				break;
			case 2:
				console.printf("\n********** Your Selection: Read A User Record **********\n");
				// us.getUser();
				break;
			case 3:
				console.printf("\n********** Your Selection: Batch Read Tweets For A User **********\n");
				// us.batchGetUserTweets();
				break;
			case 4:
				console.printf("\n********** Your Selection: Scan All Tweets For All Users **********\n");
				 // ts.scanAllTweetsForAllUsers();
				break;
			case 5:
				console.printf("\n********** Your Selection: Record UDF -- Update User Password **********\n");
				// us.updatePasswordUsingUDF();
				break;
			case 6:
				console.printf("\n********** Your Selection: Query Tweets By Username And Users By Tweet Count Range **********\n");
				// ts.queryTweets();
				break;
			case 7:
				console.printf("\n********** Your Selection: Stream UDF -- Aggregation Based on Tweet Count By Region **********\n");
				// us.aggregateUsersByTweetCountByRegion();
				break;
			case 12:
				console.printf("\n********** Create Users **********\n");
				// us.createUsers();
				break;
			case 23:
				console.printf("\n********** Create Tweets **********\n");
				// ts.createTweets();
				break;
			default:
				break;
			}
		}
	}
} catch (AerospikeException e) {
	console.printf("AerospikeException - Message: " + e.getMessage()
			+ "\n");
	console.printf("AerospikeException - StackTrace: "
					+ UtilityService.printStackTrace(e) + "\n");
} catch (Exception e) {
	console.printf("Exception - Message: " + e.getMessage() + "\n");
	console.printf("Exception - StackTrace: "
					+ UtilityService.printStackTrace(e) + "\n");
} finally {
	if (client != null && client.isConnected()) {
		// Close Aerospike server connection
		client.close();
	}
	console.printf("\n\nINFO: Press any key to exit...\n");
	console.readLine();
}
```

### Running the Application

To run the jar:

```bash
java -jar target/tweetaspike-example-application-1.0.0-full.jar
```

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
If “Connection to Aerospike cluster failed!” appears, ensure that the Aerospike Server is running and available, and confirm that **asServerIP** and **asServerPort** are properly set.
{{/note}}

### Next Step
- [Create a User and Tweet](/docs/client/java/examples/application/create.html) 

