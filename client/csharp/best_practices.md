---
title: C# Client Best Practices
description: Use these best practices with the Aerospike C# client and Aerospike database. 
categories:
  - aerospike-client-csharp
tags:
  - aerospike-client-csharp
---

Use these best practices with the Aerospike C# client and Aerospike database.

- Use `AerospikeClient` for synchronous applications.
	- Set `ClientPolicy.maxConnsPerNode` to the maximum number of connections allowed per server node. The number of connections used per node depends on how many concurrent threads issue database commands plus sub-threads used for parallel multi-node commands (batch, scan, and query). One connection will be used for each thread.  A transaction will fail if the maximum number of connections would be exceeded.

- Use `AsyncClient` for asynchronous applications.
	- Set `AsyncClientPolicy.asyncMaxCommands` to the maximum number of concurrent commands handled by the client at any point in time.

- Subscribe to the client logging facility, which sends important cluster status information messages:

	```cs
	public class MyConsole {
		public MyConsole() {
            Log.SetLevel(Log.Level.INFO);
            Log.SetCallback(LogCallback);
		}

		public void LogCallback(Log.Level level, String message)
		{
			// Write log messages to the appropriate place.
		}
	}
	```

- Each `AerospikeClient` and `AsyncClient` instance spawns a maintenance thread that periodically pings all server nodes for partition maps. Multiple client instances creates additional load on the server, so, it's best to use only one client instance per cluster and share that instance with multiple threads. `AerospikeClient` and `AsyncClient` are thread-safe.

- By default, the user-defined key is not stored on the server. The key is converted to a hash digest which is used to identify a record. Use one of the following methods to make the key must persist on the server:
    - Set `WritePolicy.sendKey` to true to send the key to the server for storage on writes, and retrieve it for multi-record scans and queries.
    - Explicitly store and retrieve the key in a bin.

- Do not use `Value` or `Bin` constructors that take in an object. These constructors are slower than hard-coded constructors, as the object must be queried for its real type. Also, they use the default C# serializer, which is the slowest of all serialization implementations. It is best to serialize the object with an optimal serializer, and use the `byte[]` constructor.

- Use `AerospikeClient.operate()` to batch multiple operations (add/get) on the same record in a single call.

- Enable Replace mode on the transaction when all record bins are created or updated in a call. In Replace mode, the server does not have to read the old record before updating, which increases performance. Do not enable Replace mode when a subset of bins is updated:

	```cs
	WritePolicy policy = new WritePolicy();
	policy.recordExistsAction = RecordExistsAction.REPLACE;
	client.put(policy, key, bins);
	```

- Each database command takes in a policy as the first argument. If the policy is identical for a group of commands, reuse the policies instead of instantiating policies for each command. To reuse policies:
	- Set `ClientPolicy` to its defaults, and pass in a NULL policy on each command:
	```cs
	ClientPolicy policy = new ClientPolicy();
	policy.readPolicyDefault.timeout = 50;	
	policy.readPolicyDefault.maxRetries = 1;
	policy.readPolicyDefault.sleepBetweenRetries = 10;
	policy.writePolicyDefault.timeout = 200;	
	policy.writePolicyDefault.maxRetries = 1;
	policy.writePolicyDefault.sleepBetweenRetries = 50;

	AerospikeClient client = new AerospikeClient(policy, "hostname", 3000);
	client.Put(null, new Key("test", "set", 1), new Bin("bin", 5));
	```

	- If a policy needs to change from the default (such as setting an expected generation), instantiate that policy on the fly:
	```cs
		public void PutIfGeneration(Key key, int generation, Bin... bins) {
			WritePolicy policy = new WritePolicy(client.writePolicyDefault);
			policy.generationPolicy = GenerationPolicy.EXPECT_GEN_EQUAL;
			policy.generation = generation;
			client.Put(policy, key, bins);
		}
	```

