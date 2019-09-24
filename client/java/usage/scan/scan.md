---
title: Scan Records
description: The Aerospike Java client provides the ability to scan all records in a specified Namespace and set. 
---

The Aerospike Java client provides the ability to scan all records in a specified Namespace and set. 

### Scanning Records

This example counts the number of records in the query result:

```java
import com.aerospike.client.AerospikeClient;
import com.aerospike.client.AerospikeException;
import com.aerospike.client.Key;
import com.aerospike.client.Record;
import com.aerospike.client.ScanCallback;
import com.aerospike.client.policy.Priority;
import com.aerospike.client.policy.ScanPolicy;
 
public final class ScanParallelTest implements ScanCallback {
    public static void main(String[] args) {
        try {
            new ScanParallelTest().runTest();
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
     
    private int recordCount;
 
    public void runTest() throws AerospikeException {
        AerospikeClient client = new AerospikeClient("127.0.0.1", 3000);
         
        try {
            ScanPolicy policy = new ScanPolicy();
            policy.concurrentNodes = true;
            policy.priority = Priority.LOW;
            policy.includeBinData = false;
 
            client.scanAll(policy, "test", "demoset", this);       
            System.out.println("Records " + recordCount);
        }
        finally {
            client.close();
        }
    }
 
    public void scanCallback(Key key, Record record) {
        recordCount++;
        if ((recordCount % 10000) == 0) {
            System.out.println("Records " + recordCount);
        }
    }
}
```

The scan policy is initialized, and if
- `concurrentNodes = true` queries nodes in parallel for records using threads; otherwise, sequentially queries each node. 
- `includeBinData = true` only returns the specified bins (or all bins by default).

This example scan policy sequentially scans all bins: 

```java
ScanPolicy policy = new ScanPolicy();
policy.concurrentNodes = true;
policy.priority = Priority.LOW;
policy.includeBinData = false;
```

The scan can now execute. If an exception is thrown, parallel scan threads to other nodes are terminated and the exception propagates back through the initiating scan call:

```java
client.scanAll(policy, "test", "demoset", this);
```

The scan returns records to the user-defined callback. Most often, multiple threads are calling your scan callback in parallel. Ensure that your scan callback implementation is thread safe. 

This example callback counts the number of records returned:

```java
public void scanCallback(Key key, Record record) {
    recordCount++;
    if ((recordCount % 10000) == 0) {
        System.out.println("Records " + recordCount);
    }
}
```
