---
title: Asynchronous Read
description: This code example initiates a database `get` call with a `RecordListener` parameter.
---

The async `get` method takes extra event loop and write listener arguments.  The listener is called when the `get()` completes.

### Initiate the Read

```java
import com.aerospike.client.async.EventLoop;
import com.aerospike.client.async.EventLoops;

// Find an event loop from eventLoops created in connect example.
EventLoop eventLoop = eventLoops.next();

// Read all the bins.
Policy policy = new Policy();
Key key = new Key("test", "myset", "mykey");
client.get(eventLoop, new ReadHandler(), policy, key);
```

### Completion Handler

When the database returns from the `read` transaction, either `onSuccess` or `onFailure` is called.

```java
private class ReadHandler implements RecordListener {
    @Override
    public void onSuccess(Key key, Record record) {
        // Read completed.
        Object received = (record == null)? null : record.getValue("mybin");    
        System.out.println(String.format("Received: " + received));         
    }
 
    @Override
    public void onFailure(AerospikeException e) {
        e.printStackTrace();
    }
}
```

[Next](/docs/client/java/usage/async/scantouch.html)
