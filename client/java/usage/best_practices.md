---
title: Java Client Best Practices
description: Follow these best practices when using the Aerospike Java client API. 
categories:
  - aerospike-client-java
tags:
  - aerospike-client-java
  - java
---

Follow these best practices when using the Aerospike Java client API.

- Use `AerospikeClient` for both synchronous and asynchronous applications.  `AsyncClient` is now obsolete and exists solely for legacy applications.  `AerospikeClient` asynchronous methods contain a new `EventLoop` argument that allows the user to specify which event loop thread should process the command.  Multiple commands specifying the same event loop can then be assumed to be single threaded and slower atomic operations are not necessary.  The old `AsyncClient` does not allow the event loop to be specified.

- Subscribe to the client logging facility to receive important cluster status messages:

	```java
	public class MyConsole implements Log.Callback {
		public MyConsole() {
			Log.setLevel(Log.Level.INFO);
			Log.setCallback(this);
		}

		@Override
		public void log(Log.Level level, String message) {
			// Write log messages to the appropriate place.
		}
	}
	```

- Set `ClientPolicy.maxConnsPerNode` to the maximum number of connections allowed (default 300) per server node. Synchronous and asynchronous connections are tracked separately.  The number of connections used per node depends on concurrent commands in progress plus sub-commands used for parallel multi-node commands (batch, scan, and query). One connection will be used for each command.  A transaction will fail if the maximum number of connections would be exceeded.

- Each `AerospikeClient` instance spawns a maintenance thread that periodically makes info requests to all server nodes for cluster status and partition maps. Multiple client instances create additional load on the server. Use only one client instance per cluster in an application and share that instance among multiple threads. `AerospikeClient` is **thread-safe**.

- By default, the user-defined key is not stored on the server. It is converted to a hash digest which is used to identify a record. If the user-defined key must persist on the server, use one of the following methods:
    - **Set `WritePolicy.sendKey` to true** &mdash; the key is sent to the server for storage on writes, and retrieved on multi-record scans and queries.
    - Explicitly store and retrieve the user-defined key in a bin.

- Do not use `Value` or `Bin` constructors that take in an object. 
	These constructors are slower than hard-coded constructors because the object must be queried (using `instanceof`) for its real type. They also use the default Java serializer, which is the slowest of all serialization implementations. 
	Instead, serialize the object with a better serializer and use the byte[] constructor.

- Use `AerospikeClient.operate()` to batch multiple operations (add/get) on the same record in a single call.

- In cases where all record bins are created or updated in a transaction, enable Replace mode on the transaction to increase performance. 
	The server then does not have to read the old record before updating. Do not use Replace mode when updating a subset of bins.

	```java
	WritePolicy policy = new WritePolicy();
	policy.recordExistsAction = RecordExistsAction.REPLACE;
	client.put(policy, key, bins);
	```

- Each database command takes in a policy as the first argument. 
	If the policy is identical for a group of commands, reuse them instead of instantiating policies for each command:
	- Set `ClientPolicy` defaults and pass in a null policy on each command.
	```java
	ClientPolicy policy = new ClientPolicy();
	policy.readPolicyDefault.socketTimeout = 50;
	policy.readPolicyDefault.totalTimeout = 110;
	policy.readPolicyDefault.sleepBetweenRetries = 10;
	policy.writePolicyDefault.socketTimeout = 200;
	policy.writePolicyDefault.totalTimeout = 450;
	policy.writePolicyDefault.sleepBetweenRetries = 50;

	AerospikeClient client = new AerospikeClient(policy, "hostname", 3000);
	client.put(null, new Key("test", "set", 1), new Bin("bin", 5));
	```

	If a policy needs to change from the default (such as setting an expected generation), instantiate that policy on-the-fly:
	```java
		public void putIfGeneration(Key key, int generation, Bin... bins) {
			WritePolicy policy = new WritePolicy(client.writePolicyDefault);
			policy.generationPolicy = GenerationPolicy.EXPECT_GEN_EQUAL;
			policy.generation = generation;
			client.put(policy, key, bins);
		}
	```
