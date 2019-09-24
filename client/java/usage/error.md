---
title: Error Handling
description: Aerospike java client error handling.
---

The java client can throw the following runtime exceptions:

Exception | Description 
--- | ---
AerospikeException                 | Base exception.  A more specific error code can be obtained by getResultCode().  [View Result Codes](https://github.com/aerospike/aerospike-client-java/blob/master/client/src/com/aerospike/client/ResultCode.java)
AerospikeException.Timeout         | Transaction has timed out.
AerospikeException.Serialize       | Error using java or messagepack encoding/decoding.
AerospikeException.Parse           | Error parsing server response.
AerospikeException.Connection      | Error establishing connection to server node.
AerospikeException.InvalidNode     | Error when chosen node is not active.
AerospikeException.ScanTerminated  | Scan was terminated prematurely.
AerospikeException.QueryTerminated | Query was terminated prematurely.
AerospikeException.CommandRejected | Async command was rejected because the maximum concurrent database commands have been exceeded.

Here is example code that handles an AersopikeException.

```java
AerospikeClient client = new AerospikeClient("192.168.1.150", 3000);

try {
	try {
		client.put(null, new Key("test", "demo", "key"), new Bin("testbin", 32));
	}
	catch (AerospikeException.Timeout aet) {
		// Handle timeouts differently.
		retryTransaction();
	}
	catch (AerospikeException ae) {
		throw new MyException("AerospikeException " + ae.getResultCode() + ": " + ae.getMessage());
	}
}
finally {
	client.close();
}
```
