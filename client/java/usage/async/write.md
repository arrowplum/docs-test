---
title: Asynchronous Write
description: This code example initiates a database `put` call with a `WriteListener` parameter.
---

The async `put()` method takes extra event loop and write listener arguments.  The listener is called when the `put()` completes.

### Initiate the Write

```java
import com.aerospike.client.async.EventLoop;
import com.aerospike.client.async.EventLoops;

// Find an event loop from eventLoops created in connect example.
EventLoop eventLoop = eventLoops.next();

// Write a single value.
WritePolicy policy = new WritePolicy();
Key key = new Key("test", "myset", "mykey");
Bin bin = new Bin("mybin", "myvalue");
client.put(eventLoop, new WriteHandler(), policy, key, bin);
```

### Completion Handler

When the database returns from the `write` transaction, either `onSuccess` or `onFailure` is called.

```java
private class WriteHandler implements WriteListener {
    @Override
    public void onSuccess(Key key) {
        try {
            // Write succeeded.  Now do something else.
            client.get(eventLoop, new ReadHandler(), policy, key);
        }
        catch (Exception e) {               
            e.printStackTrace();
        }
    }
 
    @Override
    public void onFailure(AerospikeException e) {
        e.printStackTrace();
    }
}
```

AerospikeClient methods require event loops that conform to the Aerospike EventLoop interface.  If using a Netty event loop directly, a compatible event loop can be obtained by methods on an NettyEventLoops instance.  Here is an example:

```java
import com.aerospike.client.AerospikeClient;
import com.aerospike.client.Bin;
import com.aerospike.client.Key;
import com.aerospike.client.async.EventLoop;
import com.aerospike.client.async.NettyEventLoops;
import com.aerospike.client.policy.WritePolicy;

public class MyWriter {
    private NettyEventLoops eventLoops;

    public MyWriter(NettyEventLoops eventLoops) {
        this.eventLoops = eventLoops;
    }

    public void write(io.netty.channel.EventLoop nettyLoop, AerospikeClient client, WritePolicy policy, Key key, Bin bin) {
        EventLoop eventLoop = eventLoops.get(nettyLoop);
        client.put(eventLoop, new WriteHandler(), policy, key, bin);
    }
}
```

[Next](/docs/client/java/usage/async/read.html)
